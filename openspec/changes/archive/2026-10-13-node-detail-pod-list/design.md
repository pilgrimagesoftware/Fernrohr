# Design

## Context

`fetch_object` (`object_detail/fetch.rs`) already fetches one secondary, fallible dataset
alongside the object itself - the events naming it - keeping that result as its own
`Result<Vec<_>, String>` so a permissions failure on the secondary fetch doesn't fail the whole
panel (see its doc comment: "kept apart from the object itself, as pod detail does"). `sections_for`
(`sections/mod.rs`) is otherwise a pure `(kind, object) -> Vec<ObjectSection>` dispatch with no
access to anything beyond the one object; `cluster::node` builds the Node section fields from the
typed `Node` alone. `FieldValue::References` already renders a list of objects as
`resource-links` links with no new UI work.

See proposal.md for why; see `specs/object-detail/spec.md` for the exact requirement text.

## Goals / Non-Goals

**Goals:**
- Extend the existing "secondary fetch alongside the object" pattern rather than inventing a
  second one - pods-on-node becomes the second instance of that pattern (events being the first),
  not a special case.
- Keep every other kind's `sections_for` call unaffected: only the Node arm gets new data.

**Non-Goals:**
- Live-updating the pod list while the panel stays open (no watch, matching the object panel's
  existing poll-on-open/refetch-on-reopen behavior for its primary object and events - this
  change does not add streaming to either).
- Showing per-pod status/resource usage inline in the Node panel - the pod names/links are enough;
  a pod's own detail panel is one link-follow away.

## Decisions

**Field-selector list, not a Store scan.** List pods cluster-wide with
`Api::<Pod>::all(client).list(&ListParams::default().fields(&format!("spec.nodeName={name}")))`
rather than filtering an in-memory pod `Store`: the object detail panel has no standing watch on
pods (unlike a Pods list panel), and opening one would mean a second watcher lifecycle just for
this field, which `.claude/rules/rust-structure.md`'s no-I/O-on-render-path and existing
watcher-refcounting design argue against for a value fetched once per panel-open. A field-selector
`list` is the same shape as the existing events fetch (a one-shot `kube` call keyed off the
target), not a new category of data access.

**Threading the fetched pods into `sections_for` only for Node.** `ObjectDetailState::Loaded`
gains a third field, `node_pods: Option<Result<Vec<ObjectRef>, String>>` (`None` for every
non-Node kind, so the common path's size and shape barely change), set by `fetch_object` and read
by `panel.rs` when calling `sections::sections_for`. Alternative considered: give `sections_for`
a generic "extra context" parameter every kind could use - rejected as speculative; nothing else
needs it today, and the module's own doc comment ("nothing that shows a reference to it needs to
change") argues for keeping the dispatch's signature stable for the 20 kinds that don't need this.

**`cluster::node` takes the pod list as a parameter, not a second return value.** Its signature
becomes `node(node: &Node, pods: Option<&Result<Vec<ObjectRef>, String>>) -> Vec<ObjectSection>`,
appending a `Pods` field (via `FieldValue::References`, unqualified since every entry is a Pod) or
a text field stating the list error, mirroring how the Events tab already reports a listing
failure in prose rather than failing the panel.

**`ObjectRef` built directly from the `Pod` list results, not round-tripped through
`DynamicObject`.** `kube`'s typed `Api<Pod>` already gives namespace/name directly; converting to
`ObjectRef { kind: Pod, namespace, name }` needs no intermediate dynamic representation, unlike
the main object fetch which stays dynamic to support every discovered kind generically.

## Risks / Trade-offs

[A node with hundreds of pods makes the Pods field very long] → Render it as the Lines/References
list already renders long lists elsewhere (scrollable section, not truncated) - no new truncation
UI needed; if this proves to be a real usability problem in practice, that's a follow-up change,
not a reason to hold this one.

[Field-selector list support varies slightly by `kube`/API server version for `spec.nodeName`] →
This is a standard, long-stable Kubernetes field selector for Pods (used by `kubectl get pods
--field-selector spec.nodeName=...` since early Kubernetes versions); no fallback planned.

[Permission to list pods cluster-wide may be narrower than permission to get a single Node] →
Handled explicitly per the MODIFIED requirement's third scenario: the failure surfaces as the
Pods field's own error text, not a panel-wide failure.
