# Spec Delta

## ADDED Requirements

### Requirement: Quick look at a selected pod
With a pod selected in a Pods table, the Quick Look action - Space, a "Pods: Quick Look" command, or
the row's context menu - SHALL open a popover anchored to the selected row showing the pod's name
and namespace, phase (colored, with its text), ready containers, restarts, age, node, pod IP, owner
as a link, each container's image and state, and its most recent Warning event if it has one. The
popover SHALL offer an Open Details action, by button and by Enter, that opens the pod's detail panel
and closes the popover. Space or Escape SHALL close it and return focus to the table.

#### Scenario: Opening a quick look
- **WHEN** a Pods table has focus with a crashing pod selected and the user presses Space
- **THEN** a popover beside that row shows the pod's phase, `1/2` ready, its restart count, each
  container's image and state, and the latest `BackOff` warning

#### Scenario: Scanning several pods
- **WHEN** a quick look is open and the user presses Down
- **THEN** the table's selection moves to the next pod and the popover shows that pod, without
  closing and reopening

#### Scenario: Going to the full detail
- **WHEN** a quick look is open and the user presses Enter
- **THEN** the pod's detail panel opens (or is focused if already open) and the popover closes

#### Scenario: Live while open
- **WHEN** a quick look is open and the pod's ready count changes
- **THEN** the popover updates in place

#### Scenario: The pod goes away
- **WHEN** the pod is deleted while its quick look is open
- **THEN** the popover says the pod no longer exists rather than closing silently

### Requirement: Pod rows have a context menu
Right-clicking a pod row SHALL select it and open a context menu offering Quick Look, Open Details,
Logs and YAML, each running the same command as its key.

#### Scenario: Logs from the context menu
- **WHEN** the user right-clicks a pod row and chooses Logs
- **THEN** that pod's logs open, as pressing `l` on the row would
