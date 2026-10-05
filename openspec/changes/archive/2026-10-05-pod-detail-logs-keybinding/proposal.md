# Proposal

## Why

The pod detail panel has no way to reach a pod's logs. A user who opened detail to inspect a
pod's fields or a container's state must close it, go back to the Pods list, and reselect the
row to open Logs - breaking the keyboard-first workflow every other pod-detail action follows.

## What Changes

- Add a "View Logs" command to the pod detail panel, bound to a key, scoped to the panel's key
  context (`PodDetailPanel`), so it appears in the command palette while the panel has focus.
- The command opens the Logs panel for the pod the detail panel is scoped to, reusing the
  existing `SelectedPod` / `ShowLogs` nav path (`App/app/src/ui/nav.rs`,
  `App/app/src/k8s/resource/pods/selection.rs`) rather than inventing a second route to Logs.
- The container list handed to Logs comes from the pod's `spec.containers` names, same as the
  Pods list row already does, so multi-container pods still default to the first container and
  let the user switch (per the existing `pod-logs` container-selection requirement).

## Capabilities

### New Capabilities

(none)

### Modified Capabilities

- `pod-detail`: adds a requirement that the panel can open the pod's logs via a keyboard-reachable
  command, in addition to its existing field/YAML/tab actions.

## Impact

- `App/app/src/k8s/resource/pod_detail/commands.rs`: new action, key constant, command
  registration.
- `App/app/src/k8s/resource/pod_detail/panel.rs`: new action handler that sets `SelectedPod` and
  dispatches `ShowLogs`.
- `App/app/src/k8s/resource/pod_detail/render.rs` (or wherever panel hints render): show the new
  shortcut in the key-hint row.
- No changes to the Logs panel itself or `pod-logs` capability - it already handles container
  selection.
