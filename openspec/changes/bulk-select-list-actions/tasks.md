# Tasks

## 1. Shared checked-set selection model

- [ ] 1.1 Add a checked-set type (`HashSet` keyed by pod uid for `pods/`, by `ObjectRef` for
  `object_list/`) to each panel's state, independent of the existing single-cursor
  `TableState::selected_row`. Verify with `cargo build`.
- [ ] 1.2 Render a checkbox cell per row reflecting checked state, toggled by click. Verify
  visually with `cargo run` against a test cluster: clicking a checkbox checks/unchecks the row
  without moving the cursor.
- [ ] 1.3 Add a `ToggleRowChecked` action bound to `x` on the focused row (both panels; not
  Space, which `k9s-remaining-keybindings` already bound to `pods.quick_look` on the Pods table -
  see design.md), registered as a command scoped to each panel's `KeyContext`. Add a test
  simulating the `x` keystroke and asserting the focused row's checked state flips.
- [ ] 1.4 Add a `CheckRange` action bound to Shift+`x` (and Shift+click on the checkbox),
  checking every row between the last-checked row and the target inclusive, following the
  namespace-set editor's shift-range model. Add a test covering a 3-7 range check.
- [ ] 1.5 Ensure checked identities survive a watch-driven row reorder/update (reuse the existing
  reselect-by-identity logic `pods_table::reselect` already applies to the single cursor). Add a
  test: check 2 rows, apply a watch event that reorders the table, assert the same 2 objects stay
  checked.

## 2. Bulk action bar and eligibility rules

- [ ] 2.1 Add a bulk action bar component shown whenever the checked set is non-empty, listing
  only actions valid for every checked row's kind:
  - Delete (reusing `resource_actions::delete` from `k9s-remaining-keybindings`, looped over the
    checked set - not a new delete call), Label, Annotate, Copy Name(s), Copy YAML - always
    offered, any kind.
  - Restart Rollout, Rollback Rollout - offered when every checked row is a Deployment,
    DaemonSet, or StatefulSet, in any mix of those three.
  - Scale - offered when every checked row is a Deployment, StatefulSet, or ReplicaSet, in any
    mix of those three.
  - Cordon, Uncordon, Drain - offered only when every checked row is a Node.
  - View Logs - offered only when every checked row is a Pod.
  Verify with a test per eligibility rule (a kind outside a group hides that group's actions; a
  mix within a group still offers them; a mixed-kind selection outside every group still offers
  Delete/Label/Annotate/Copy).
- [ ] 2.2 Make the bar's actions reachable from the keyboard and the command palette while a list
  panel with a non-empty checked set has focus, each as a registered command with a
  `keymap.toml`-overridable binding. Give `ToggleRowChecked`/`CheckRange` their `x`/Shift-`x`
  defaults from Task 1; give the bar's other actions (Delete, Restart Rollout, Rollback Rollout,
  Scale, Cordon, Uncordon, Drain, Label, Annotate, Copy Name(s), Copy YAML, bulk View Logs) an
  empty default binding (`default_binding: ""`, the existing pattern `ui/table_fit.rs` and
  `ui/panel/tabs.rs` use for a command with no obvious single key) so none of them default onto
  `ctrl-d`/`ctrl-k` or any other key already bound in `PodsPanel`'s or `ObjectListPanel`'s context -
  each stays reachable via the bar's click target, the palette, and whatever key a user assigns in
  `keymap.toml`. Verify each command appears in the palette under test, and that the keymap
  conflict checker reports no collisions against the default keymap.

## 3. Shared confirmation, input prompt, and batch-outcome UI

- [ ] 3.1 Add a small shared confirmation component parameterized by verb, count, kind(s), and an
  optional value ("Delete 4 Pods?", "Restart Rollout for 3 Deployments?", "Scale 2 Deployments to
  3 replicas?", "Set label team=platform on 1 Pod, 1 ConfigMap?", "Open 8 Logs panels?"),
  following the tunnel editor's inline confirm-flag pattern rather than a new modal primitive.
  Verify with a snapshot/unit test per action's confirmation text.
- [ ] 3.2 Add a shared one-or-two-field input prompt (replica count for Scale; key and value for
  Label/Annotate), with inline validation (Scale rejects non-numeric or negative input), that
  leads into the confirmation on submit. Verify with a test covering valid input proceeding to
  confirmation and invalid input showing an inline error without proceeding.
- [ ] 3.3 Add a batch-outcome report (per-object name + ok/error, with the error reason) shown
  after any bulk mutating action completes. Verify with a test where a mocked batch partially
  fails and the report names the one that failed and why.

## 4. Mutating calls: Delete, Restart Rollout, Rollback Rollout

- [ ] 4.1 Add a shared module (e.g. `k8s::resource::bulk_actions`) for the bulk-only calls
  (restart/rollback/scale/cordon/drain/label-annotate below); for Delete, call
  `k8s::resource::resource_actions::delete` (the same helper `pods.delete`/`pods.kill` already
  use) per checked object rather than adding a second delete call. Verify with a test against the
  mock cluster (`k8s::test_cluster`) that bulk Delete issues the same request shape single-row
  Delete does, for both a Pod and a non-Pod kind.
- [ ] 4.2 Add the rollout-restart call: patches a Deployment's, DaemonSet's, or StatefulSet's
  `spec.template.metadata.annotations["kubectl.kubernetes.io/restartedAt"]` to the current
  timestamp, matching `kubectl rollout restart`'s own mechanism - one function parameterized by
  `ObjectRef`/`DiscoveredKind`, no per-kind branching needed since the patch path is identical
  for all three. Verify with a test per kind (Deployment, DaemonSet, StatefulSet) asserting the
  annotation is set to a fresh RFC3339 timestamp on each call.
- [ ] 4.3 Add the rollout-rollback call: reads the kind's `ControllerRevision` history, finds the
  revision before the current one, and reapplies its template, matching `kubectl rollout undo`.
  Verify with a test asserting the object's template matches the previous revision's after the
  call, and a second test asserting a clean per-object failure when no previous revision exists.
- [ ] 4.4 Add a batch-runner that calls the per-object function for each checked object and
  collects a `Vec<(ObjectRef, Result<(), Error>)>`, used by both panels and by every bulk action.
  Verify with a test where one of several calls is made to fail and the collected result reflects
  it individually.

## 5. Mutating calls: Scale, Cordon/Uncordon, Drain

- [ ] 5.1 Add the scale call: patches `spec.replicas` to the given count for Deployment,
  StatefulSet, or ReplicaSet. Verify with a test per kind asserting the replica count after the
  call matches the requested value.
- [ ] 5.2 Add the cordon/uncordon call: patches a Node's `spec.unschedulable` to `true`/`false`.
  Verify with a test asserting the field's value after each call.
- [ ] 5.3 Add the drain call: cordons the node, lists pods with a field-selector on
  `spec.nodeName`, filters out pods owned by a DaemonSet or with no controller reference (mirror
  pods), and calls `Api::evict` on the rest. Verify with a test asserting a DaemonSet-owned pod
  and a mirror pod are skipped while an evictable pod is evicted, and a second test asserting a
  PodDisruptionBudget-rejected eviction is reported as a per-pod failure without aborting the
  node's other evictions or un-cordoning it.

## 6. Mutating calls: Label/Annotate, and read-only Copy actions

- [ ] 6.1 Add the label/annotate call: one function taking a flag for which map
  (`metadata.labels` or `metadata.annotations`), a key, and an optional value, patching any
  `Api<DynamicObject>`; an empty value removes the key instead of setting it. Verify with a test
  setting a label, a second test setting an annotation, and a third test removing an existing
  key via an empty value.
- [ ] 6.2 Add Copy Name(s) and Copy YAML, reading each checked row's already-loaded object data
  (no cluster call) and writing to the system clipboard - newline-joined names, and `---`-joined
  manifests respectively. Verify with a test asserting clipboard content for a 3-object checked
  set, and that neither action shows a confirmation or produces a batch-outcome report.

## 7. Wire every action into both panels

- [ ] 7.1 Wire Delete: show the confirmation, run the batch delete via the shared runner on
  confirm, show the outcome report, clear the checked set for every object that succeeded. Verify
  with a keystroke-driven test: check 2 pods, Delete, confirm, assert both delete requests were
  issued against the mock cluster. Add a second test asserting canceling issues no requests and
  leaves every checked row checked.
- [ ] 7.2 Wire Restart Rollout and Rollback Rollout (Deployment/DaemonSet/StatefulSet only), each
  through the shared confirm → run → report flow. Verify with a test per action covering a
  single-kind and a mixed-kind (Deployment+StatefulSet) checked set.
- [ ] 7.3 Wire Scale (Deployment/StatefulSet/ReplicaSet only) through the input prompt →
  confirmation → run → report flow. Verify with a test asserting the entered replica count is
  applied to every checked object.
- [ ] 7.4 Wire Cordon, Uncordon, and Drain (Node only), each through the shared confirm → run →
  report flow. Verify with a test per action against a mock cluster with Nodes, including Drain's
  PDB-rejection case surfacing in the outcome report.
- [ ] 7.5 Wire Label and Annotate through the input prompt → confirmation → run → report flow, for
  any checked kind. Verify with a test applying a label across a mixed-kind checked set (e.g. a
  Pod and a ConfigMap).
- [ ] 7.6 Wire Copy Name(s) and Copy YAML with no confirmation step. Verify with a test asserting
  immediate clipboard content with no confirmation UI shown.
- [ ] 7.7 Wire View Logs (Pods only) to open one Logs panel per checked pod directly when the
  checked count is 5 or fewer, and to show a count confirmation first above that threshold.
  Verify with a test checking 3 pods (opens directly, no confirmation) and a test checking 8 pods
  (confirms first; canceling opens none, confirming opens 8).

## 8. Final checks

- [ ] 8.1 Run `cargo fmt && cargo clippy -- -D warnings && cargo test` and confirm all pass.
- [ ] 8.2 Manually exercise the full flow against a test cluster (`cargo run`): multi-select with
  mouse and keyboard in both the Pods table and a non-Pod list panel; run every bulk action
  (Delete, Restart Rollout and Rollback Rollout against each of Deployment/DaemonSet/StatefulSet,
  Scale against each of Deployment/StatefulSet/ReplicaSet, Cordon/Uncordon/Drain against Nodes,
  Label/Annotate against a mixed-kind selection, Copy Name(s)/Copy YAML, and View Logs above and
  below the 5-panel threshold); confirm confirmation and outcome reporting behave as specced for
  each.
