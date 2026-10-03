# Proposal

## Why

Checking one thing about a pod - is it ready, how many restarts, what image, which node - means
opening a full detail panel, which takes over a dock slot and has to be closed again. Scanning
several pods this way is slow. A quick look that shows the essentials over the list, and gets out of
the way, covers the common case without leaving the table.

## What Changes

- **Quick Look**: with a pod selected in a Pods table, Space (also a "Pods: Quick Look" command and
  a new row context menu item) opens a popover anchored to the row with the pod's most important
  details.
- **Contents**: name and namespace; phase (colored, text kept); ready containers and restarts; age;
  node and pod IP; owner (as a link); each container's image and state; and the most recent Warning
  event, if any.
- **While open**: Up/Down move the table's selection and the popover follows, so several pods can be
  scanned without reopening it; Space or Escape closes it; an "Open Details" button (Enter) opens the
  pod's detail panel and closes the popover.
- **Row context menu**: right-clicking a pod row offers Quick Look, Open Details, Logs and YAML - the
  panel-scoped pod commands that left the menu bar.

## Capabilities

### New Capabilities

(none)

### Modified Capabilities

- `resource-browser`: the Pods table gains a quick-look popover and a row context menu.

## Impact

- App: Pods panel (new popover view, key bindings, context menu) reusing the pod detail's field
  projection and status colors from `resource-detail-ui-improvements`, and the pod event watch from
  `pod-events-time-window` for the latest warning.
