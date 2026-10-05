# Design

## Context

Both list panels today hold exactly one cursor position via gpui-kit's `TableState<Delegate>`
(`SelectionMode::Row`, one `selected_row: Option<usize>`) - there is no multi-row concept in the
component library itself (checked across `gpui-kit` 0.7's `crates/component/src/table/state.rs`).
Each panel's delegate owns its rows: `pods/` keyed by pod uid, `object_list/` by `ObjectRef`
(kind + namespace + name) over `Api<DynamicObject>`. Single-row actions (describe, YAML, open)
are registered commands scoped to each panel's `KeyContext`, dispatched the same way across both
panels (`object_list/commands.rs`, `pods/commands.rs`).

`k9s-remaining-keybindings` already shipped the app's first cluster-mutating calls: single-row
delete (`ctrl-d`, confirmed), force-kill (`ctrl-k`, no confirmation), YAML edit (server-side
apply), and shell/exec. Delete's confirm-before-mutate pattern and its
`resource_actions::delete(client, kind, name, namespace, force)` helper are this change's closest
and most direct precedent - Bulk Delete reuses both outright (looping the helper over the checked
set) rather than inventing a second delete path. The tunnel editor's `confirming_delete: bool`
field + inline confirm UI (`ui/tunnels/editor/actions.rs`) remains the precedent for the bulk
confirmation component's inline-flag shape, though that one deletes local config, not a cluster
object. Restart/rollback rollout, scale, cordon/drain, and label/annotate are the actions here
that are genuinely new mutating calls.

`kube-client` 4.2's `Api::evict` already wraps the eviction subresource
(`api/subresource.rs`), so Drain needs no raw-request workaround - it lists the pods scheduled
on a node (field-selector on `spec.nodeName`, same kind of list call the app already makes
elsewhere), filters out DaemonSet-owned and mirror pods, and calls `evict` on the rest.

## Goals / Non-Goals

**Goals:**
- One checked-set model, one bulk-action-bar component, and one set of mutating calls, shared by
  both the Pods table and `ObjectListPanel` rather than built twice.
- Checked state lives beside, not instead of, the existing single-cursor selection - every
  existing single-row command keeps working unchanged.
- Every destructive or disruptive action always confirms; Copy Name(s)/Copy YAML never do (they
  mutate nothing); View Logs confirms only once the checked count passes a threshold, since
  opening panels isn't destructive, just potentially disruptive at scale.
- A failure on one object in a batch doesn't hide or cancel the outcome of the rest.
- Each mutating action reuses `kubectl`'s own mechanism for the same operation (the
  restart/rollback annotation and revision history, the eviction subresource, `spec.replicas`,
  `spec.unschedulable`), so this app's bulk actions behave the same as the equivalent `kubectl`
  command a user already knows, not a reinvented approximation of it.

**Non-Goals:**
- A fully generic "run any action in bulk" plugin framework. Nine actions, each with its own
  eligibility rule, share one checked-set model, one confirmation component, and one
  batch-runner; that's enough structure to avoid duplicating plumbing without building a
  framework no action outside this change will ever use.
- Undo for Delete, Scale, Cordon/Uncordon, Label/Annotate, or Drain. Only rollout
  restart/rollback has an undo-shaped counterpart in Kubernetes itself (revision history); the
  others are real cluster operations the same as their `kubectl` equivalents, and recovery is
  whatever the cluster itself offers.
- Draining onto a specific replacement node, or any node-maintenance workflow beyond cordon and
  evict. This change gives Drain the same scope `kubectl drain` has by default.

## Decisions

- **Checked set as `HashSet<ObjectRef>` (or pod uid) held per panel, not per row.** Mirrors
  `pods/selection.rs`'s existing identity choice (uid for Pods) and `object_list`'s `ObjectRef`
  for everything else, so no new identity type is needed, and checked state survives a watch-driven
  row reorder the same way the single selection already needs to (`pods_table::reselect`'s
  existing re-anchor-by-identity logic is reused, not reinvented, for the checked set).
- **A thin checkbox affordance bolted onto the existing row render, not a second selection mode
  in gpui-kit.** The library's `SelectionMode` stays `Row` for cursor/focus; checked-ness is
  app-level state rendered as a checkbox cell and read on click/`x`, same pattern as the
  Collapsible disclosure rows in `pod_detail`. Keeps the change inside the app, with no upstream
  gpui-kit dependency bump needed.
- **Checkbox toggle binds to `x` (Shift+`x` for range), not Space.** k9s itself uses Space to mark
  a resource for its own bulk operations, but the Pods table already binds Space to
  `pods.quick_look` (from `pod-quick-look`), so Space isn't free here. `x`
  is unused in both `PodsPanel`'s and `ObjectListPanel`'s key contexts today (checked against
  every command registered in `pods/commands.rs` and `object_list/commands.rs`); Shift+`x` for
  range-select then just extends the same key the way Shift already extends `w` into
  `WarpAllToNamespace`'s `shift-w`. The mouse route (click a checkbox, Shift-click to extend) is
  unaffected either way.
- **The bulk action bar's own actions (Delete, Restart Rollout, Rollback Rollout, Scale,
  Cordon, Uncordon, Drain, Label, Annotate, Copy Name(s), Copy YAML, bulk View Logs) default to no
  key binding, reachable via the bulk action bar itself and the command palette.** This is an
  established pattern for registered commands without an obvious single key (`default_binding:
  ""`, as `ui/table_fit.rs`, `ui/panel/tabs.rs` and others already do) - not a gap, since the
  command palette and the bar's own click target already give each a keyboard and mouse route
  (`.claude/rules/keyboard-first.md`). It also sidesteps any ambiguity with
  `k9s-remaining-keybindings`'s single-row `ctrl-d` (`pods.delete`) and `ctrl-k` (`pods.kill`):
  those two keys keep acting on the focused row alone, in their existing context, whether or not
  other rows are checked; bulk Delete never shares their key, so there's no question of which
  target set `ctrl-d` affects. `keymap.toml` still lets a user bind any of them to a key of their
  choice.
- **One shared crate-internal module for the mutating calls** (`k8s::resource::bulk_actions` or
  similar), taking a `ClusterConnection` + a list of `ObjectRef`s/`DiscoveredKind`, doing the
  per-kind call (delete, restart/rollback patch, scale patch, cordon/uncordon patch, drain's list
  + evict, label/annotate patch) per object, and returning a per-object `Result`. Both panels
  call into this instead of each growing their own mutating code - this is also where each
  action's `kubectl`-equivalent mechanism lives, so it's defined once per action rather than
  once per panel.
- **Restart and Rollback Rollout match `kubectl rollout restart`/`undo` exactly, for all three
  kinds they support.** Deployment, DaemonSet, and StatefulSet are each `kubectl rollout
  restart`/`undo`-eligible kinds. Restart patches
  `spec.template.metadata.annotations["kubectl.kubernetes.io/restartedAt"]` to now, identical
  across all three. Rollback reads the kind's `ControllerRevision` history (same as `kubectl
  rollout undo`) and reapplies the previous revision's template. One function per action,
  parameterized only by `ObjectRef`/`DiscoveredKind`, covers all three kinds; there's no
  kind-specific branching to write, and no new mechanism for this app to invent or diverge on.
- **Scale patches `spec.replicas` directly**, the same field `kubectl scale` patches, for
  Deployment, StatefulSet, and ReplicaSet - all three expose it at the same path, so again one
  function covers all three.
- **Cordon/Uncordon and Drain match `kubectl cordon`/`uncordon`/`drain`.** Cordon/Uncordon patch
  `spec.unschedulable`. Drain cordons first, then lists pods on the node (field-selector on
  `spec.nodeName`) and calls `Api::evict` on each evictable one, skipping pods owned by a
  DaemonSet or with no controller (mirror pods) - the same default `kubectl drain` applies. A
  PodDisruptionBudget rejecting one eviction fails only that pod's entry in the batch outcome;
  it does not stop evicting the node's other pods or un-cordon the node.
- **Label/Annotate share one patch function** differing only in whether it writes to
  `metadata.labels` or `metadata.annotations`, and apply to any kind via
  `Api<DynamicObject>::patch` - no kind restriction, since every Kubernetes object carries both
  maps.
- **Copy Name(s) and Copy YAML read already-loaded row data, with no cluster call at all.** Each
  list panel already holds its rows' `DynamicObject`s (or `Pod`s) for rendering; these two
  actions just serialize what's already in memory to the clipboard, so they carry no RBAC
  surface, no confirmation, and no batch outcome to report.
- **A shared one-field input prompt for Scale (replica count) and Label/Annotate (key, value).**
  Same small modal shape - one or two text inputs, Enter to proceed to confirmation, Escape to
  cancel - reused rather than building a one-off dialog per action.
- **Confirmation is a shared small component (count + kind(s) + action + optional value), not a
  dialog per action.** Delete, Restart/Rollback Rollout, Scale, Cordon/Uncordon, Drain,
  Label/Annotate, and the >5 View Logs case all just need "you are about to X N <kind(s)> [to
  <value>] - continue?"; one component parameterized by verb/count/kind/value covers all of them,
  following the tunnel editor's inline-flag-driven confirm rather than introducing a new modal
  primitive per action.
- **Batch outcome reported as a simple per-object list (name, ok/error), not aggregated into one
  pass/fail.** Every mutating call here fails independently per object; collapsing that to one
  boolean would hide which object needs attention.

## Risks / Trade-offs

- This ships six distinct *new* mutating actions at once (restart/rollback rollout, scale,
  cordon/uncordon, drain, label/annotate - Delete reuses `k9s-remaining-keybindings`'s existing
  call). Getting patch/evict error handling (RBAC-denied, already-gone, conflict, PDB-blocked)
  right here sets the pattern every future bulk action will follow - worth the extra care the
  tasks give it, not something to rush to ship the UI faster.
- A checked set keyed by identity (uid / `ObjectRef`) rather than row index means a checked
  object that disappears from a live watch mid-batch simply drops out of the checked set's
  practical effect (the mutating call for it will 404/fail rather than corrupt a different row) -
  acceptable, and consistent with how the single-cursor selection already behaves on live
  updates.
- Drain is the riskiest single action here - it affects every pod on a node, not just the
  checked rows themselves, and its PDB/DaemonSet-skip rules are easy to get subtly wrong. It gets
  its own task group and its own explicit PDB-rejection test rather than being folded into the
  generic batch-runner tests.
- Rollback Rollout depends on `ControllerRevision` history existing and being readable; a cluster
  that has pruned history or restricts reading `ControllerRevision`s makes rollback fail
  per-object the same way any other RBAC-denied call does - no special-cased error message
  beyond what the batch outcome already reports.
- Scale and Label/Annotate's one-field prompt adds a step beyond a plain confirm, but there is no
  safe default replica count or label value to assume - the trade-off is one extra keystroke for
  correctness.
- Every destructive or disruptive action here acts on real infrastructure; the always-confirm
  rule is the primary safeguard. No "are you sure" fatigue mitigation (e.g. typing the count to
  confirm) is planned for v1 - revisit if usage shows it's needed.
