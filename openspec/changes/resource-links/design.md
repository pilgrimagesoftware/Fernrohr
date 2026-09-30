# Design

## Context

Current state (see proposal.md for why this matters):

- `NavTarget` has three variants: `Kind(DiscoveredKind)` (a list panel), `Logs`, and
  `Pod(PodRef)` (one pod's detail). Only core `v1 Pod` has a concrete panel
  (`nav::has_concrete_panel`). Every other discovered kind opens `PlaceholderPanel`.
- `MainWindow::open_target_with_view` (`util/shell.rs`) is where opening happens, and it already
  dedups: it builds a `PanelKey` from the `PanelScope` (target + `context_name` + namespaces) and
  focuses a matching open panel instead of adding another. It always scopes to the *window's*
  `context_name`.
- Pod detail's `PodFieldValue::Link(String)` is only colored text. Owners are comma-joined into
  one string. Volumes are formatted strings in a `List`. Env sources and image pull secrets aren't
  projected at all.
- Pod detail opening is driven by a dataless `ShowPodDetail` action that reads the `SelectedPod`
  global. That works for "the selected row" but not for "this particular reference".

## Goals / Non-Goals

**Goals:**
- One typed reference representation, one "does this kind have a viewer" decision, and one render
  helper, shared by every detail view present and future.
- Following a reference reuses the existing open/dedup/focus path, not a second one.

**Non-Goals:**
- **New viewers.** This change adds no Node, ConfigMap, ReplicaSet (etc.) panel. Consequence: on
  the day it lands, the only live links from pod detail are Namespace, and any reference that
  resolves to a Pod. Everything else shows as plain text until its viewer exists. A follow-up
  generic single-object viewer (metadata + YAML for any discovered kind) would make every reference
  live at once. It's deliberately a separate change, because it is its own capability with its own
  fetch/watch and RBAC questions.
- Opening in a new window or split (modifier-click). Opening follows the normal open path.
- Links out of the YAML view. Only the structured view has fields to link.
- Showing secret values. Following a Secret link opens whatever the Secret viewer shows once it
  exists; this change fetches nothing through a link.

## Decisions

### A typed `ObjectRef`, built at projection time

`ObjectRef { group: String, kind: String, namespace: Option<String>, name: String }`, built in
the projection (`pod_fields` and its helpers) from the API objects' own reference shapes:
`OwnerReference` (`apiVersion` → group), `spec.nodeName` (core `Node`, cluster-scoped),
`spec.serviceAccountName`, volume sources, `envFrom`/`valueFrom`, `imagePullSecrets`. It's parsed
once at the boundary, so render code never re-derives a kind from a formatted string.

- **No version.** Identity for "which viewer" is `(group, kind)`. Owner references carry a
  version, but a viewer is per kind, and matching on version would make a
  `apps/v1` vs `apps/v1beta2` difference stop a link from resolving.
- **Namespace filled in at projection.** A namespaced object referencing another by bare name
  (every pod reference except Node) gets the pod's namespace. Cluster-scoped kinds (Node) get
  `None`. The render side never reasons about scope.
- *Alternative considered:* keep `Link(String)` and parse `Kind/name` back out when clicked.
  Rejected. It's the "validate, don't parse" shape that loses group and namespace, and the
  owner comma-join already shows how such strings drift.

`PodFieldValue::Link(String)` becomes `PodFieldValue::References(Vec<ObjectRef>)` (one entry for
Node/Namespace/Service Account, several for owners). Volume rows and container cards carry
`Vec<ObjectRef>` beside their text instead of folding the name into it.

### One predicate decides linkability: `nav::viewer_for`

`fn viewer_for(&ObjectRef) -> Option<NavTarget>` in `ui/nav.rs` replaces `has_concrete_panel` as
the single place that knows which kinds have panels. Today it resolves core `Pod` →
`NavTarget::Pod`, and `Namespace` → the Pods list scoped to that namespace. `None` means plain
text. A new viewer is registered by extending this one function, which is how the spec's "a kind
gains a viewer" scenario holds without touching any reference site.

- *Alternative considered:* a `NavTarget::Object(ObjectRef)` variant that absorbs `Pod(PodRef)`
  now. Deferred. With one concrete object viewer, that abstraction has no second caller yet.
  The first non-Pod viewer is when to fold `Pod` into it.

### Opening goes through `open_target_with_view`, scoped to the source panel

A shared `ui/link.rs` renders an `ObjectRef` as a link (if `viewer_for` says so) or plain text.
Activating a link dispatches a data-carrying action
`FollowReference { context_name, target: ObjectRef }` that `MainWindow` handles by calling
`open_target_with_view` with that `context_name`, not the window's. That makes the dedup/focus
behaviour in the spec the existing behaviour rather than new logic.
`open_target_with_view` gains an explicit context parameter; the window's own context becomes the
default at its current call sites. This matters now: `1-window-context-bar` introduces
multi-context windows, where the window's context and a panel's context can differ.

- *Alternative considered:* set a global "selected reference" and dispatch a dataless action, as
  `ShowPodDetail` does with `SelectedPod`. Rejected: a link is its own selection, and a shared
  global would race between two panels.

### Keyboard: a "Go to…" picker per detail view

`g` (panel key context, shown in the hint bar as "Go to…") opens a filterable list of that view's
followable references: kind, name and the field they came from, with Enter to follow. That covers
"reachable by keyboard" without making dozens of links tab stops (the panels deliberately use
`tab_stop(false)` on in-body controls). It's built once in `ui/link.rs` from the same
`Vec<ObjectRef>` the view renders, so the picker and the visible links can't disagree.

- *Alternative considered:* Tab/arrow focus cycling through links in place. Rejected for now:
  focus is already hard to see (see `per-tab-close-button`), and a pod with many env sources
  would make cycling slow.

### Env references deduped per container

`envFrom` (`configMapRef`/`secretRef`) and `valueFrom` (`configMapKeyRef`/`secretKeyRef`) collapse
to one reference per object per container, in first-seen order, so twenty keys from one ConfigMap
read as one link. Projected-volume sources (`projected.sources[].configMap/secret`) are volume
references like any other.

## Risks / Trade-offs

- [Most links render as plain text at first, so the feature looks small] → Deliberate (no dead
  links). Say so in the PR, and propose the generic single-object viewer next.
- [`pod_detail.rs` is ~2,600 lines against a 500-line limit, and this change adds to it] → Split it
  first, in its own PR after `1-window-context-bar` merges (that branch edits the same file). The
  reference projection goes into its own module either way.
- [`open_target_with_view` signature change conflicts with `1-window-context-bar`] → Sequence
  after it merges. The change is additive (a context argument with the window's as default).
- [A reference to a kind whose group isn't in discovery (CRD removed)] → `viewer_for` returns
  `None`, so it shows as plain text, same as any unviewable kind.
