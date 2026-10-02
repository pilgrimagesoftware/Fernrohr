# Tasks

## 1. Generic list panel

- [x] 1.1 Move 401 detection and the reference-counted watch registry out of `pods/` into a shared module keyed by `(context, ApiResource)`; Pods use it unchanged. Verify existing Pods watch tests pass.
- [x] 1.2 Add `k8s/resource/object_list/` watch and store over `Api<DynamicObject>`, applying watcher events to rows. Verify with store unit tests for apply/delete/restart events, and that a 403 initial list surfaces as an error state.
- [x] 1.3 Add the `ObjectListPanel` table with base columns (Name, Namespace for namespaced kinds only, Age), plural title, namespace selector (hidden for cluster-scoped kinds), filter, sort, column resize/reorder and keyboard hints matching the Pods panel. Verify with panel tests for cluster-scoped vs namespaced columns and sorting.
- [x] 1.4 Route `NavTarget::Kind` in `ui/nav.rs` to `ObjectListPanel` for every non-Pod kind, remove `has_concrete_panel`, and keep `PlaceholderPanel` only for restoring an undiscovered kind. Verify with a nav test that Deployments, Nodes and a CRD each open a list panel.
- [x] 1.5 Activating a row (Enter, double-click) opens `NavTarget::Object`, focusing an already-open panel. Verify with a test that a second activation focuses rather than duplicates.
- [x] 1.6 Save and restore `ObjectListPanel` (kind, namespace, column layout) in `util/shell/panels.rs`. Verify with a restore round-trip test.

## 2. Per-kind list columns (needs 1)

- [ ] 2.1 Add `object_list/columns.rs` with the column-definition type and `Cell` variants (Text, Number, Ratio, Age) with numeric sorting, computing cells once per watch event. Verify with unit tests for sort order of Number and Ratio cells.
- [ ] 2.2 Add Workloads columns: Deployment, ReplicaSet, StatefulSet, DaemonSet, Job, CronJob. Verify each with a fixture-object extractor test.
- [ ] 2.3 Add Config and Network columns: ConfigMap, Secret, Service, Ingress, Endpoints, EndpointSlice, NetworkPolicy. Verify each with a fixture-object extractor test.
- [ ] 2.4 Add Storage, Cluster and Access Control columns: PersistentVolumeClaim, PersistentVolume, StorageClass, Node, Namespace, ServiceAccount, RoleBinding, ClusterRoleBinding. Verify each with a fixture-object extractor test, including a malformed object yielding empty cells rather than a panic.

## 3. Object-detail sections (independent of 1 and 2)

- [x] 3.1 Add `sections/network.rs`: Service, Ingress, Endpoints, EndpointSlice and NetworkPolicy sections, with backend Services, TLS Secrets and target Pods as references. Verify with section tests in `object_detail/tests/sections.rs`.
- [x] 3.2 Extend `storage.rs` with PersistentVolume (claim and StorageClass as references) and StorageClass (including the default-class annotation). Verify with section tests.
- [x] 3.3 Extend `workloads.rs` with CronJob (active Jobs as references) and add the Namespace section. Verify with section tests.
- [x] 3.4 Add `sections/rbac.rs`: Role and ClusterRole rules, and RoleBinding and ClusterRoleBinding role and subjects as references. Verify with section tests.
- [x] 3.5 Register every new kind in the `sections/mod.rs` dispatch. Verify with a test that each listed kind returns a non-empty section set.

## 4. Integration verification

- [ ] 4.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` pass in the App workspace.
- [ ] 4.2 Manual smoke test against a real cluster: open Deployments, Services, ConfigMaps, Nodes, PVCs, RoleBindings and one CRD from the Resource panel; confirm live rows, per-kind columns and sorting; open a row of each and confirm its detail sections and links; restart the app and confirm the list panels restore.
