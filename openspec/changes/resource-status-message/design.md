# Design

## Context

See proposal.md - Why. Two facts from the current code shape this design:

- **List columns are a static table.** `object_list/columns.rs::for_kind(group, kind)` returns
  `Option<&'static KindColumns>`, matched on a fixed list of built-in `(group, kind)` pairs. A kind
  not in that list - every CRD, Flux's `HelmRelease`/`Kustomization` included - gets `None` and
  shows only the base Name/Namespace/Age columns. There is no `additionalPrinterColumns` support of
  any kind today, so there is nothing yet to collide with.
- **Detail sections are a static dispatch too.** `object_detail/sections/mod.rs::sections_for`
  matches the same kind of fixed `(group, kind)` list and falls back to `None` (Overview only) for
  anything else. Kinds that are in the list already render `status.conditions` as colored badges via
  `sections/common.rs::condition_badges`, but none of them also show a plain-language message -
  badges name the condition type, not what it says.

**FreeLens finding.** Confirmed via Exa web search against the FreeLens source and the Flux CRDs
(2026-10-07):
- FreeLens's `kube-object-conditions-list.tsx` renders `status.conditions`, sorting the `Ready`
  condition first and showing its `.message`
  (https://github.com/freelensapp/freelens/blob/92fd2e39/packages/core/src/renderer/components/kube-object-conditions/kube-object-conditions-list.tsx).
- The older Lens Flux extension computes the list message the same way:
  `obj.status?.conditions?.find(c => c.type === 'Ready')?.message`
  (https://github.com/okaufmann/lens-extension-fluxcd/commit/56a6553ec297c9a54b8675353d293ad149828a0b).
- Flux's own CRDs declare this as a printer column, so `kubectl get` shows it without any
  client-side special-casing: both `helmreleases.helm.toolkit.fluxcd.io` and
  `kustomizations.kustomize.toolkit.fluxcd.io` define
  `additionalPrinterColumns: [{name: Status, jsonPath: '.status.conditions[?(@.type=="Ready")].message'}]`
  (https://github.com/fluxcd/helm-controller/blob/main/config/crd/bases/helm.toolkit.fluxcd.io_helmreleases.yaml,
  https://github.com/fluxcd/kustomize-controller/blob/690c8c8a/config/crd/bases/kustomize.toolkit.fluxcd.io_kustomizations.yaml).
  Note the column's own title there is **"Status," not "Message"** - naming ours "Message" avoids a
  same-content, same-name collision if Fernrohr later adds generic `additionalPrinterColumns`
  rendering (a Non-Goal here; see below).

This confirms the task's assumption and rules out `lastAppliedRevision` or a Flux-specific field as
the source.

## Goals / Non-Goals

**Goals:**
- One condition-reading derivation, shared by the list column and the detail section, that works
  for any kind - built-in or CRD - without Flux-specific code.
- Surface it in both surfaces per proposal.md, reusing the existing `Cell::Status` /
  `FieldValue::Status` / `BadgeTone` machinery rather than adding new rendering primitives.
- Keep the message's tone meaningful but conservative: only color it confidently when the chosen
  condition is `Ready` (whose polarity - `True` is good - is universal). A fallback, non-`Ready`
  condition's polarity cannot be assumed (plenty of Kubernetes conditions are "bad when `True`"), so
  it is shown untoned.

**Non-Goals:**
- Generic `additionalPrinterColumns` rendering (reading a CRD's declared printer columns and
  rendering all of them). This change covers exactly one derived column; broader printer-column
  support is a larger, separate change and is left for later.
- Reading the CRD's OpenAPI schema to decide column presence up front (see Decision 2). That would
  need new discovery plumbing (fetching and parsing `CustomResourceDefinition` schemas) this change
  does not add.
- Flux-specific parsing (revision strings, Helm release names, etc.) - only the generic
  type/status/reason/message/lastTransitionTime condition shape is read.

## Decisions

### Decision 1: One shared derivation function in `status_tone`

Add `pub fn condition_message(conditions: &[Value]) -> Option<(String, Tone)>` (or an equivalent
small struct) to `app/src/k8s/resource/status_tone.rs`, operating on the raw JSON conditions array
(`serde_json::Value`), not a typed `k8s-openapi` `Condition` - Flux's CRDs have no generated Rust
type, so a JSON-level reader is the only one that works for both built-in and CRD kinds alike.

Algorithm:
1. Find the condition whose `type == "Ready"`.
   - If found: the message is its `.message` (empty message => `None`, same "empty is absent"
     convention `Cell::text`/`Cell::status` already use). Tone: `status == "True"` -> `Good`;
     `status == "False"` -> `Bad`; anything else (`Unknown`, missing, or a non-standard value) ->
     `Warning`.
2. Else, pick the condition with the latest parseable `lastTransitionTime` that has a non-empty
   `.message`. Tone: always `Neutral` - the polarity of an arbitrary condition type cannot be
   inferred, so it is shown without implying good or bad.
3. No conditions, or no condition with a non-empty message either way: `None` - the column/field
   simply doesn't appear, matching how `Cell::status` and `ObjectField` already treat "nothing to
   show."

**Alternatives considered:** per-kind Flux-specific readers (rejected - proposal requires
generic, non-Flux-specific logic, and FreeLens's own approach generalizes trivially); always
preferring the latest-transitioned condition regardless of `Ready` (rejected - would make healthy
objects display something other than the Ready message the moment any other condition updates,
diverging from what FreeLens and `kubectl` both show).

### Decision 2: List column presence is decided from what the panel has loaded, not from CRD schema

`for_kind` stays a static table for the built-in kinds it already covers - no change there, and no
Message column is added alongside their existing status-bearing column (Job's `Status`, Node's
`Status`, etc.), so there is never a kind with two overlapping status cells.

For a kind with **no** static `KindColumns` entry (every CRD today), the list panel adds a single
generic Message column once it has observed that kind's objects carry a non-empty
`status.conditions` array - checked against the objects already in the panel's live store (the same
`Entity` the table renders from), not a separate fetch. In practice: the column is present once the
first object of that kind with conditions arrives; a kind with zero objects, or whose objects never
set `status.conditions`, shows no Message column, matching today's base-columns-only behavior
exactly.

**Alternatives considered:**
- *Always add the column for any kind without a hardcoded table, empty when absent.* Rejected: most
  CRDs have no `status.conditions` at all, so this would add a permanently empty column to nearly
  every CRD kind's list, and `resource-browser` has no column-hide affordance to let a user remove
  it.
- *Resolve it from the CRD's declared OpenAPI schema (`status.conditions` as a schema property).*
  This is the "correct" long-term answer and mirrors how `kubectl`/FreeLens effectively get it for
  free from `additionalPrinterColumns`, but Fernrohr's discovery (`k8s::cluster::discovery`) only
  calls the API discovery endpoint today (kind, plural, verbs) and never fetches a CRD's schema.
  Adding that is a reasonable follow-up (tracked as a Non-Goal above) but is out of proportion to
  this change.

### Decision 3: Detail panel changes land in two places

1. **Kinds that already render Conditions badges** (Deployment, DaemonSet, StatefulSet via
   `sections/common.rs::condition_badges`) gain one additional `ObjectField` - `"Message"`, built
   from the same typed object's `status.conditions` via `condition_message` - inserted immediately
   before the existing `"Conditions"` field, not replacing it. The badges still name every
   condition; the message adds what the Ready one says.
2. **Kinds with no hardcoded section** (`sections_for`'s `_ => None` arm - CRDs, including Flux
   kinds) gain a new generic fallback: read `status.conditions` from the object's raw JSON (no typed
   struct needed, same reasoning as Decision 1), and if any condition has a usable message, return
   one `ObjectSection` titled "Status" with a `"Message"` field and a `"Conditions"` badges field
   (built by generalizing `condition_badges` to accept raw `(type, status)` pairs read from JSON
   instead of a caller-typed iterator - it already takes `impl IntoIterator<Item = (&str, &str)>`,
   so this needs no signature change, only a JSON-sourced caller). An object with no conditions at
   all keeps today's Overview-only panel - this is strictly additive.

The Message field uses the existing `FieldValue::Status { text, tone }` / `ObjectField::status(...)`
path, so it renders, copies (per `object-detail`'s existing "Values can be copied" requirement, which
the spec delta extends), and participates in live updates exactly as every other status field does -
no new rendering code, no new `FieldValue` variant.

### Decision 4: Keyboard and selection - no new affordances needed

- **List column:** the cell is a normal `Cell::Status`, so it truncates, sorts, resizes and
  reorders like any other column. The full message is reachable by keyboard the same way any other
  truncated cell's detail already is: select the row and press `d` (describe) to open the object's
  detail panel, where the full text is shown and copyable. This satisfies keyboard-first without a
  new truncation/tooltip widget - the existing "describe" route already gets a keyboard user to the
  full text.
- **Detail panel:** the Message field is copyable via the existing copy-control convention
  (`object-detail` - "Values can be copied"), reachable on hover and by keyboard exactly as
  ConfigMap values and container images already are.
- No panel-content focus frame and no hover-only affordance are introduced; both surfaces reuse
  existing, already-compliant patterns.

## Risks / Trade-offs

- **A CRD with conditions but an empty-message Ready condition shows nothing** → acceptable: this
  matches `Cell::status`'s existing "empty text is absent" convention and avoids showing a
  misleadingly blank badge.
- **The list column can appear or disappear as a panel's first conditioned object arrives or a
  freshly opened panel is still empty** → mitigated by recomputing presence from the live store
  (not a one-time snapshot), so it still settles to the correct state once the watch delivers data;
  documented in tasks.md as a case to test explicitly.
- **Fallback (non-`Ready`) messages are always untoned** → intentional per Decision 1; a future
  change could special-case well-known condition types (e.g. Flux's `Stalled`) if this proves too
  conservative in practice.
- **No generic `additionalPrinterColumns` support** → out of scope; Non-Goals above. If added later,
  the "Message" vs. Flux's own "Status" column-title mismatch (noted in Context) avoids a literal
  duplicate, but the two would show the same content under different names - worth revisiting
  together at that time.
