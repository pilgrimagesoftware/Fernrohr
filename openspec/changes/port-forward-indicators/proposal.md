# Proposal

## Why

A port-forward started from the Pods list (`shift-f`) shows its result as a notice in the list
panel itself. That's a poor fit:

- The notice isn't tied to the pod it describes.
- It disappears from view when you move on.
- There's no way to see, from the pod, that it is being forwarded.
- Stopping the forward means going to Manage Tunnels.

Mouse users also have no way to start a forward at all, because it's keyboard-only.

## What Changes

- **Pods list:** a new **Forwards** column. A pod with active forwards shows a forward icon with a
  count, and its tooltip lists each `127.0.0.1:<local> → <container port>`. The panel-level
  success notice goes away. Failures appear as a transient notification, not a panel notice.
- **Pod detail panel:** while the pod has active forwards, a compact strip above the tabs shows
  each forward's local address, with icon buttons to copy the address and to stop the forward.
- **Containers tab:** next to each declared container port, an icon button starts a forward of
  that port. Once forwarded, the port shows its local address, with copy and stop icon buttons.
- **Buttons:** every new button is an icon with a tooltip, not a text label. Each has a matching
  registered command, so the keyboard reaches everything:
  - `shift-f` already starts a forward.
  - A new **Stop Port Forward** command stops one. If the pod has several forwards, it asks which.

  These commands work in the Pods list and in the pod detail panel.
- Services get the same Forwards column in their list.

## Capabilities

### New Capabilities

- `port-forward-indicators`: where an active port-forward is shown, and how it is started,
  copied and stopped from the pod row, the pod detail panel and the container ports.

### Modified Capabilities

(none: the managed-forward requirements in `k9s-remaining-keybindings` stay as written. This change
adds the in-context surfaces.)

## Impact

- `App/app/src/k8s/cluster/port_forwards.rs` (the app-wide forward list): expose each pod's or
  service's active forwards as an observable lookup, so rows and panels re-render when forwards
  start or stop.
- `App/app/src/k8s/resource/pods/` (columns, render, commands), `object_list/` for Services, and
  `pod_detail/` (header strip, container card ports, commands).
- Removes the Pods list's port-forward success notice. Errors go to the app's notification toast.
