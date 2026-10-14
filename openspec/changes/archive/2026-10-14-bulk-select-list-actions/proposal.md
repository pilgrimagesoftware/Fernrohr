# Proposal

## Why

Every list panel (Pods table, the generic `ObjectListPanel` every other kind opens) only ever
acts on one row at a time - open, describe, YAML, and, since `k9s-remaining-keybindings`, delete
(`ctrl-d`, with confirmation), force-kill (`ctrl-k`, no confirmation), edit, shell, and
port-forward on the one selected row. There is still no way to delete several objects at once,
restart several Deployments, drain several nodes, or tail several pods' logs at once; each
requires opening and acting on one row after another. k9s and kubectl both treat multi-object
bulk action as a basic workflow, and this app has none of it yet.

This is not, however, the app's first mutating path - `k9s-remaining-keybindings` already shipped
single-row delete, force-kill, YAML edit (server-side apply), and shell/exec. Bulk Delete here
reuses that single-row delete's confirmation dialog and its `resource_actions::delete` backend
call (looping it over the checked set) rather than inventing a second delete mechanism; the other
bulk actions (restart/rollback rollout, scale, cordon/drain, label/annotate) are the genuinely new
mutating surface this change adds.

## What Changes

- Add checkbox-style multi-row selection to every list panel (Pods table and
  `ObjectListPanel`), toggled by `x` on the focused row and by clicking a row's checkbox,
  with Shift+click/Shift+`x` range selection following the same model as the namespace-set
  editor's range select. (Not Space: the Pods table already binds Space to Quick
  Look on the Pods table, so checkbox toggle needs its own key - see design.md.)
- Add a bulk action bar/menu, visible whenever one or more rows are checked, offering the
  actions that make sense for what's checked:
  - **Delete** - any kind, any number of checked rows; reuses the single-row Delete command's
    confirmation dialog and `resource_actions::delete` call from `k9s-remaining-keybindings`,
    looped over the checked set, not a second delete mechanism.
  - **Restart rollout** - only offered when every checked row is one of the three workload kinds
    `kubectl rollout restart` itself supports: Deployment, DaemonSet, or StatefulSet. A checked
    set may mix those three kinds freely, since the restart mechanism is identical for all of
    them.
  - **Rollback rollout** (undo to previous revision) - the counterpart to Restart Rollout, same
    kind eligibility (Deployment, DaemonSet, StatefulSet), using each kind's `ControllerRevision`
    history the same way `kubectl rollout undo` does.
  - **Scale** - only offered when every checked row is a Deployment, StatefulSet, or ReplicaSet;
    prompts for one replica count applied to every checked object.
  - **Cordon / Uncordon** - only offered when every checked row is a Node; toggles
    `spec.unschedulable`.
  - **Drain** - only offered when every checked row is a Node; cordons each node and evicts its
    evictable pods (respecting PodDisruptionBudgets, skipping DaemonSet-managed and mirror pods,
    matching `kubectl drain`'s own default behavior).
  - **Label / Annotate** - any kind, any number of checked rows; prompts for one key and value
    applied (or removed, if the value is left empty) across every checked object.
  - **Copy Name(s)** and **Copy YAML** - any kind, any number of checked rows; non-destructive,
    no confirmation. Copy Name(s) joins the checked objects' names with newlines; Copy YAML joins
    their manifests separated by `---`.
- **BREAKING (behavioral): the app can now mutate cluster objects**, which it could not do
  before. Delete, Restart Rollout, Rollback Rollout, Scale, Cordon/Uncordon, Drain, and
  Label/Annotate each issue a real request against the cluster.
- Destructive or disruptive actions (Delete, Restart Rollout, Rollback Rollout, Scale,
  Cordon/Uncordon, Drain, Label/Annotate) always show a confirmation naming the count, kind, and
  (where relevant) the value being set before running, and report per-object success/failure
  rather than failing silently if one object in the batch errors.
- View Logs opens one Logs panel per checked pod, but asks for confirmation first when that would
  open more than 5 panels at once, so a big multi-select doesn't flood the dock unannounced.
- Checking any row is keyboard-reachable and shown as a registered command (palette entry,
  `keymap.toml`-overridable), consistent with every other list-panel action
  (`.claude/rules/keyboard-first.md`).

## Capabilities

### New Capabilities

- `bulk-object-actions`: multi-row selection in a list panel and the batch actions (delete,
  restart rollout, rollback rollout, scale, cordon/uncordon, drain, label/annotate, view logs,
  copy name(s), copy YAML) that act on the checked set, including their confirmation rules.

### Modified Capabilities

(none - selection and action plumbing is new; existing list-panel requirements such as single-row
describe/YAML/open are unaffected and keep working exactly as today alongside the new checkboxes.)

## Impact

- `App/app/src/k8s/resource/pods/` and `App/app/src/k8s/resource/object_list/`: new per-panel
  checked-set state, a checkbox column/affordance, Shift-range selection, and a bulk action bar.
- New shared module for the mutating calls themselves (delete, rollout-restart patch, rollout-undo
  patch, scale patch, cordon/uncordon patch, drain's evict calls, label/annotate patch) over
  `Api<DynamicObject>`, callable from either panel so the two don't duplicate the cluster-call
  logic.
- New shared confirmation-and-batch-result UI (count/kind/value summary, per-object outcome),
  reusable from both panels rather than built twice.
- A new small input prompt (one text field) shared by Scale (replica count) and Label/Annotate
  (key and value), reusable rather than building two one-off dialogs.
- `App/app/src/k8s/resource/pods/commands.rs` and `object_list/commands.rs`: new actions and key
  bindings (toggle-check, select-range, open bulk menu, run each bulk action), registered the
  same way existing panel commands are.
- No change to the Logs panel or `pod-logs` capability - bulk View Logs opens N existing
  single-pod Logs panels, it does not change what one Logs panel does.
