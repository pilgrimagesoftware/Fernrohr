# Design

## Context

Pods have a hand-built list stack: a typed `kube_runtime::watcher` (`k8s/resource/pods/watch.rs`),
a store (`pods/store.rs`), a panel (`pods/panel.rs`) and a table with a closed `PodColumn` enum
(`pods_table.rs`). `ui/nav.rs::has_concrete_panel` sends every other kind to
`ui/placeholder.rs`. The generic object-detail panel (`k8s/resource/object_detail/`) already
renders any kind, and dispatches kind-specific sections in `sections/mod.rs`, but only `viewer_for`
link-following reaches it.

## Goals / Non-Goals

**Goals:**
- One list implementation that serves every non-Pod kind, built-in or CRD.
- Per-kind columns are data (a column table per kind), not per-kind panels.
- Detail sections for the kinds the spec delta adds, following the existing section modules.

**Non-Goals:**
- Moving Pods onto the generic list. Pods keep their typed stack. Merging them is a later
  refactor once the generic one has settled.
- CRD `additionalPrinterColumns`. CRDs get base columns only for now.
- Editing, deleting or scaling objects from a list.

## Decisions

### D1: A single `DynamicObject` list stack, not typed watchers per kind
New module `k8s/resource/object_list/` with watch, store, panel and table, parallel to `pods/`.
It watches `Api<DynamicObject>` using the `ApiResource` from discovery, so one implementation
covers every kind, including CRDs. *Alternative:* a typed `k8s-openapi` watcher per kind. That
means about 20 copies of the Pods stack and nothing for CRDs. Rejected.

### D2: Columns come from a per-kind column table
`object_list/columns.rs` maps a `(group, kind)` to a list of column definitions. Each one is a
header plus an extractor `fn(&DynamicObject) -> Cell`, where `Cell` is `Text`, `Number`, `Ratio`
(ready/desired, sorts by the first value) or `Age`, so sorting is numeric where the spec requires
it. Built-in extractors deserialize the object's data into the `k8s-openapi` type once per watch
event, at the moment the store stores the row, rather than on every render. Kinds missing from
the table get only the base columns (Name, Namespace, Age). Columns are identified by a string
key, not by position, so the existing resize and reorder persistence keeps working.

### D3: Same shared-watch lifecycle as Pods
A registry keyed by `(context, ApiResource)` with reference counting, generalized from the Pods
registry. The 401 detection in `pods/watch.rs::is_unauthorized` is already generic over the
watcher error type and moves to a shared place so both stacks use it. A 403 on the initial list
becomes a panel error state (spec: "A kind the user cannot list").

### D4: Routing
In `ui/nav.rs`, `NavTarget::Kind` opens the Pods panel for core/Pod and `ObjectListPanel` for
everything else. `has_concrete_panel` goes away. `PlaceholderPanel` is left only for restoring a
saved panel whose kind is no longer discovered. `util/shell/panels.rs` restores `ObjectListPanel`
from its saved kind and namespace.

### D5: Row activation reuses existing targets
Enter or double-click dispatches `NavTarget::Object` (or `NavTarget::Pod` for Pods, which is
already the case), so focusing an already-open panel comes from the existing open path for free.

### D6: Detail sections as new sibling modules
`sections/network.rs` covers Service, Ingress, Endpoints, EndpointSlice and NetworkPolicy;
`sections/rbac.rs` covers Role, ClusterRole, RoleBinding and ClusterRoleBinding. `storage.rs`
gains PersistentVolume and StorageClass, `workloads.rs` gains CronJob, and Namespace goes into
`node.rs`, renamed `cluster.rs` if it reads better. Cross-references use the existing
`resource-links` reference type. Each section gets tests in `object_detail/tests/sections.rs`.

## Risks / Trade-offs

- [More concurrent watches] → watches start only when a panel opens and stop with the last one
  (D3). That's the same cost per kind as Pods.
- [Large kinds such as Events or EndpointSlices on big clusters] → the table already virtualizes
  rows. Column cells are computed once per watch event (D2), not per frame.
- [Deserializing into typed structs can fail on unusual API versions] → if it fails, the
  kind-specific cells show empty and the base columns still render. Never panic on a bad object.
- [Two stacks (Pods and generic) drifting] → the shared watch registry and 401 handling (D3) keep
  the lifecycle code in one place. Only the table types differ.
