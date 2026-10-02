# Tasks

## 1. Generic list panel

- [x] 1.1 Move 401 detection and the reference-counted watch registry out of `pods/` into a shared module keyed by `(context, ApiResource)`; Pods use it unchanged. Verify existing Pods watch tests pass.
- [x] 1.2 Add `k8s/resource/object_list/` watch and store over `Api<DynamicObject>`, applying watcher events to rows. Verify with store unit tests for apply/delete/restart events, and that a 403 initial list surfaces as an error state.
- [x] 1.3 Add the `ObjectListPanel` table with base columns (Name, Namespace for namespaced kinds only, Age), plural title, namespace selector (hidden for cluster-scoped kinds), filter, sort, column resize/reorder and keyboard hints matching the Pods panel. Verify with panel tests for cluster-scoped vs namespaced columns and sorting.
- [x] 1.4 Route `NavTarget::Kind` in `ui/nav.rs` to `ObjectListPanel` for every non-Pod kind, remove `has_concrete_panel`, and keep `PlaceholderPanel` only for restoring an undiscovered kind. Verify with a nav test that Deployments, Nodes and a CRD each open a list panel.
- [x] 1.5 Activating a row (Enter, double-click) opens `NavTarget::Object`, focusing an already-open panel. Verify with a test that a second activation focuses rather than duplicates.
- [x] 1.6 Save and restore `ObjectListPanel` (kind, namespace, column layout) in `util/shell/panels.rs`. Verify with a restore round-trip test.

## 2. Per-kind list columns (needs 1)

- [x] 2.1 Add `object_list/columns.rs` with the column-definition type and `Cell` variants (Text, Number, Ratio, Age) with numeric sorting, computing cells once per watch event. Verify with unit tests for sort order of Number and Ratio cells.
- [x] 2.2 Add Workloads columns: Deployment, ReplicaSet, StatefulSet, DaemonSet, Job, CronJob. Verify each with a fixture-object extractor test.
- [x] 2.3 Add Config and Network columns: ConfigMap, Secret, Service, Ingress, Endpoints, EndpointSlice, NetworkPolicy. Verify each with a fixture-object extractor test.
- [x] 2.4 Add Storage, Cluster and Access Control columns: PersistentVolumeClaim, PersistentVolume, StorageClass, Node, Namespace, ServiceAccount, RoleBinding, ClusterRoleBinding. Verify each with a fixture-object extractor test, including a malformed object yielding empty cells rather than a panic.

## 3. Object-detail sections (independent of 1 and 2)

- [x] 3.1 Add `sections/network.rs`: Service, Ingress, Endpoints, EndpointSlice and NetworkPolicy sections, with backend Services, TLS Secrets and target Pods as references. Verify with section tests in `object_detail/tests/sections.rs`.
- [x] 3.2 Extend `storage.rs` with PersistentVolume (claim and StorageClass as references) and StorageClass (including the default-class annotation). Verify with section tests.
- [x] 3.3 Extend `workloads.rs` with CronJob (active Jobs as references) and add the Namespace section. Verify with section tests.
- [x] 3.4 Add `sections/rbac.rs`: Role and ClusterRole rules, and RoleBinding and ClusterRoleBinding role and subjects as references. Verify with section tests.
- [x] 3.5 Register every new kind in the `sections/mod.rs` dispatch. Verify with a test that each listed kind returns a non-empty section set.

## 5. Keyboard and titles from the first smoke test

- [x] 5.1 Bind describe (`d`) and YAML (`y`) on every list panel, panel-scoped, as registered palette commands shown in the hint row, matching the Pods table's keys. Verify with a window-level test that `d` and `y` on a selected Services row open its detail (YAML view for `y`).
  App#89: `object_list.describe` (`d`) and `object_list.yaml` (`y`), scoped to `ObjectListPanel &&
  !Input`, in Navigate and the hint row. `OpenListedObject` carries the view, and an open object
  detail switches to it. Verified by
  `util::shell::follow::list_keys_tests::down_then_d_describes_the_first_row_and_y_shows_its_yaml`
  and `up_then_y_opens_the_last_rows_yaml`.
- [x] 5.2 Down/Up on a focused list with no selection selects the first/last visible row, for the object list and the Pods table. Verify with a window-level keystroke test for each.
  App#89: the table only binds and handles Up/Down in its own element, so a list panel focused as
  a whole reached no row. `ui::list_keys` binds them in each list's context and steps the table
  from the panel's capture phase. Verified by the two tests above (object list) and
  `k8s::resource::pods::panel::list_keys_tests` (Pods table).
- [x] 5.3 A custom resource list's tab and title bar show the plural kind only, with the API group in the tab tooltip. Verify with a test on a CRD kind's title and tooltip.

## 4. Integration verification

- [x] 4.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` pass in the App workspace.
  Verified 2026-10-02 on App develop e5a36d8 (all sections plus follow-ups): fmt and clippy clean, 775 passed, 2 ignored.
- [x] 4.2 Manual smoke test against a real cluster: open Deployments, Services, ConfigMaps, Nodes, PVCs, RoleBindings and one CRD from the Resource panel; confirm live rows, per-kind columns and sorting; open a row of each and confirm its detail sections and links; restart the app and confirm the list panels restore.
  Passed 2026-10-02 (user, real cluster), on the second run. The first run found the CRD tab
  group, describe keys and arrow selection (section 5), plus unrelated layout and focus bugs fixed
  in App#86 and App#88.
