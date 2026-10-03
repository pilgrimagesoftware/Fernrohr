# Spec Delta

## Purpose

Lets a user check several rows in any list panel and run one action - delete, restart or
rollback a rollout, scale, cordon/uncordon or drain a node, label or annotate, view logs, or
copy name(s)/YAML - against all of them at once, instead of repeating a single-row action one
object at a time.

## ADDED Requirements

### Requirement: Rows are individually checkable

Every list panel (the Pods table and every `ObjectListPanel`) SHALL let the user check and
uncheck individual rows, independent of which row the single-row cursor is on, by clicking a
row's checkbox and by pressing Space on the focused row. Checked state SHALL be visually distinct
from the cursor/focus highlight.

#### Scenario: Checking a row does not move the cursor

- **WHEN** the user checks a row other than the currently focused one
- **THEN** the focused row stays focused and the checked row shows as checked

### Requirement: Range selection by Shift

Shift+click on a row's checkbox, and Shift+Space on the focused row, SHALL check every row
between the last row the user checked and the target row, inclusive, the same range-selection
model the namespace-set editor's shift-click uses.

#### Scenario: Shift-click extends from the last check

- **WHEN** the user checks row 3, then Shift-clicks row 7's checkbox
- **THEN** rows 3 through 7 are all checked

### Requirement: A bulk action bar appears while any row is checked

Whenever one or more rows in a list panel are checked, the panel SHALL show a bar or menu listing
the actions available for the checked set, and SHALL offer no action that does not apply to
every checked row's kind.

#### Scenario: Restart is hidden for a selection outside the restartable kinds

- **WHEN** the checked rows include a Deployment and a Service
- **THEN** the bulk action bar does not offer Restart Rollout

#### Scenario: Restart is offered for a mix of restartable kinds

- **WHEN** the checked rows include a Deployment and a StatefulSet, and nothing else
- **THEN** the bulk action bar offers Restart Rollout

#### Scenario: View Logs is hidden outside the Pods table

- **WHEN** rows are checked in an `ObjectListPanel` for a non-Pod kind
- **THEN** the bulk action bar does not offer View Logs

#### Scenario: Cordon, Uncordon and Drain are offered only for Nodes

- **WHEN** every checked row is a Node
- **THEN** the bulk action bar offers Cordon, Uncordon and Drain
- **WHEN** any checked row is not a Node
- **THEN** none of Cordon, Uncordon or Drain is offered

#### Scenario: Scale is offered only for scalable kinds

- **WHEN** the checked rows include a Deployment and a ReplicaSet, and nothing else
- **THEN** the bulk action bar offers Scale
- **WHEN** the checked rows include a Pod
- **THEN** the bulk action bar does not offer Scale

#### Scenario: Delete, Label, Annotate, Copy Name(s) and Copy YAML are always offered

- **WHEN** the checked rows mix several unrelated kinds (e.g. a Pod, a ConfigMap, and a Service)
- **THEN** the bulk action bar still offers Delete, Label, Annotate, Copy Name(s) and Copy YAML

### Requirement: Bulk Delete

The bulk action bar's Delete action SHALL be available for any checked set regardless of kind,
SHALL issue a delete request for each checked object, and SHALL always show a confirmation
naming how many objects of which kind(s) will be deleted before issuing any request.

#### Scenario: Confirming a bulk delete

- **WHEN** the user checks 4 Pods and chooses Delete
- **THEN** a confirmation names "4 Pods" and only issues delete requests once the user confirms

#### Scenario: Canceling leaves objects untouched

- **WHEN** the user opens the delete confirmation and cancels it
- **THEN** no delete request is sent and every checked row remains

### Requirement: Bulk Restart Rollout

The bulk action bar's Restart Rollout action SHALL be offered only when every checked row is a
Deployment, DaemonSet, or StatefulSet, SHALL patch each checked object to trigger a rollout
restart, and SHALL always show a confirmation naming how many objects of which kind(s) will be
restarted before issuing any request.

#### Scenario: Confirming a bulk restart

- **WHEN** the user checks 3 Deployments and chooses Restart Rollout
- **THEN** a confirmation names "3 Deployments" and only issues the restart patches once
  confirmed

#### Scenario: Confirming a mixed-kind bulk restart

- **WHEN** the user checks 2 Deployments and 1 DaemonSet and chooses Restart Rollout
- **THEN** a confirmation names the restart covering both kinds (e.g. "2 Deployments, 1
  DaemonSet") and only issues the restart patches once confirmed

### Requirement: Bulk Rollback Rollout

The bulk action bar's Rollback Rollout action SHALL be offered only when every checked row is a
Deployment, DaemonSet, or StatefulSet, SHALL revert each checked object to its previous revision
using that kind's `ControllerRevision` history (the same mechanism `kubectl rollout undo` uses),
and SHALL always show a confirmation naming how many objects of which kind(s) will be rolled
back before issuing any request.

#### Scenario: Confirming a bulk rollback

- **WHEN** the user checks 2 StatefulSets and chooses Rollback Rollout
- **THEN** a confirmation names "2 StatefulSets" and only reverts them once confirmed

#### Scenario: Rollback fails cleanly with no prior revision

- **WHEN** a checked object has no previous revision to roll back to
- **THEN** that object is reported as failed in the batch outcome, and every other checked
  object with a previous revision is still rolled back

### Requirement: Bulk Scale

The bulk action bar's Scale action SHALL be offered only when every checked row is a Deployment,
StatefulSet, or ReplicaSet, SHALL prompt for one replica count, SHALL apply that same replica
count to every checked object, and SHALL always show a confirmation naming the count and how
many objects of which kind(s) will be scaled before issuing any request.

#### Scenario: Scaling several objects to the same count

- **WHEN** the user checks 2 Deployments, chooses Scale, and enters 3
- **THEN** a confirmation names "scale 2 Deployments to 3 replicas" and only patches them once
  confirmed

#### Scenario: Scale rejects a non-numeric or negative input

- **WHEN** the user enters a value that is not a non-negative integer into the Scale prompt
- **THEN** the prompt shows an inline error and does not proceed to confirmation

### Requirement: Bulk Cordon and Uncordon

The bulk action bar's Cordon and Uncordon actions SHALL be offered only when every checked row
is a Node, SHALL set or clear `spec.unschedulable` on each checked Node, and SHALL always show a
confirmation naming how many Nodes will be cordoned or uncordoned before issuing any request.

#### Scenario: Confirming a bulk cordon

- **WHEN** the user checks 2 Nodes and chooses Cordon
- **THEN** a confirmation names "2 Nodes" and only marks them unschedulable once confirmed

### Requirement: Bulk Drain

The bulk action bar's Drain action SHALL be offered only when every checked row is a Node, SHALL
cordon each checked Node and then evict its evictable pods - respecting PodDisruptionBudgets and
skipping DaemonSet-managed and mirror pods, matching `kubectl drain`'s own default behavior -
and SHALL always show a confirmation naming how many Nodes will be drained before issuing any
request.

#### Scenario: Confirming a bulk drain

- **WHEN** the user checks 2 Nodes and chooses Drain
- **THEN** a confirmation names "2 Nodes" and only cordons and evicts their pods once confirmed

#### Scenario: A PodDisruptionBudget blocks one eviction

- **WHEN** draining a Node would violate a PodDisruptionBudget for one of its pods
- **THEN** that pod's eviction is reported as failed in the batch outcome, and the Node is still
  cordoned and its other evictable pods are still evicted

### Requirement: Bulk Label and Annotate

The bulk action bar's Label and Annotate actions SHALL be available for any checked set
regardless of kind, SHALL prompt for one key and one value, SHALL apply that key/value as a
label or annotation (respectively) to every checked object, SHALL remove the key instead when
the value is left empty, and SHALL always show a confirmation naming the key, value (or
"remove"), and how many objects of which kind(s) will be changed before issuing any request.

#### Scenario: Applying a label across a mixed-kind selection

- **WHEN** the user checks a Pod and a ConfigMap, chooses Label, and enters `team=platform`
- **THEN** a confirmation names setting `team=platform` on "1 Pod, 1 ConfigMap" and only patches
  them once confirmed

#### Scenario: Empty value removes the key

- **WHEN** the user chooses Annotate, enters an existing annotation's key with an empty value,
  and confirms
- **THEN** that annotation is removed from every checked object that had it

### Requirement: Bulk Copy Name(s) and Copy YAML

The bulk action bar's Copy Name(s) action SHALL copy every checked object's name to the
clipboard, one per line. The bulk action bar's Copy YAML action SHALL copy every checked
object's manifest to the clipboard, each separated by a `---` document divider. Neither action
SHALL show a confirmation or mutate anything.

#### Scenario: Copying several names at once

- **WHEN** the user checks 3 Pods and chooses Copy Name(s)
- **THEN** the clipboard holds the 3 pods' names, one per line, with no confirmation shown

### Requirement: Bulk View Logs confirms above a threshold

The bulk action bar's View Logs action SHALL be offered only when every checked row is a Pod,
and SHALL open one Logs panel per checked Pod. When the checked set has more than 5 Pods, it
SHALL show a confirmation naming the count before opening any panel; at 5 or fewer, it SHALL open
the panels directly without confirming.

#### Scenario: Small selection opens without confirming

- **WHEN** the user checks 3 Pods and chooses View Logs
- **THEN** 3 Logs panels open with no confirmation shown

#### Scenario: Large selection confirms first

- **WHEN** the user checks 8 Pods and chooses View Logs
- **THEN** a confirmation names "8" before any Logs panel opens, and canceling opens none

### Requirement: Batch actions report per-object outcome

After a bulk Delete, Restart Rollout, Rollback Rollout, Scale, Cordon, Uncordon, Drain, Label, or
Annotate request completes, the panel SHALL report which checked objects succeeded and which
failed, rather than treating the whole batch as succeeded or failed as one unit when only some
objects errored.

#### Scenario: One object fails in a larger batch

- **WHEN** a bulk delete of 5 Pods succeeds for 4 and is rejected for 1 (e.g. already deleted)
- **THEN** the panel reports that 4 succeeded and names the 1 that failed and why

### Requirement: Checking and bulk actions are keyboard-reachable

Checking a row, Shift-range selection, opening the bulk action bar, and each bulk action SHALL be
reachable from the keyboard, SHALL each be a registered command with a default binding and a
`keymap.toml` override, and SHALL appear in the command palette while the panel has focus.

#### Scenario: Running a bulk action without a mouse

- **WHEN** the user checks several rows with Space and Shift+Space alone
- **THEN** the bulk action bar's actions are reachable and operable without touching the mouse
