# Spec Delta

## Purpose

Gives a panel's tab a right-click (and keyboard-reachable) menu of tab-scoped actions, so closing,
splitting out, or pinning a tab doesn't require reaching for the command palette or dragging.

## ADDED Requirements

### Requirement: Open the tab context menu
A panel tab SHALL open a context menu scoped to that tab on right-click, and the focused tab
SHALL open the same menu in response to a keybound action, so the menu is reachable without a
mouse.

#### Scenario: Right-click opens the menu
- **WHEN** the user right-clicks a panel's tab
- **THEN** the application opens a context menu scoped to that tab at the pointer location

#### Scenario: Keyboard shortcut opens the menu for the focused tab
- **WHEN** the user invokes the open-tab-menu command while a tab is focused
- **THEN** the application opens the same context menu scoped to the focused tab

### Requirement: Tab context menu actions
The tab context menu SHALL list, in order: Close Tab, Close Other Tabs, Close Tabs to the Right,
Close All Tabs, Move to New Window, and Pin Tab (or Unpin Tab when the tab is already pinned).
Each item SHALL run the corresponding command-registry action, so every item is also independently
reachable from the command palette and rebindable in the keymap.

#### Scenario: Selecting an item runs its command
- **WHEN** the user selects an item in the tab context menu
- **THEN** the application runs that item's registered command against the tab the menu was
  opened for

#### Scenario: Items reflect command-registry state
- **WHEN** the tab context menu is open
- **THEN** each item's label matches its command's current title (e.g. "Unpin Tab" once the tab
  is pinned)

### Requirement: Items disable when they would have no effect
"Close Other Tabs" and "Close All Tabs" SHALL be disabled when the tab's group has no other
unpinned tabs. "Close Tabs to the Right" SHALL be disabled when the tab is already the rightmost
tab in its group.

#### Scenario: Only tab in its group
- **WHEN** the context menu opens for the only tab in a group
- **THEN** "Close Other Tabs" and "Close All Tabs" are shown disabled

#### Scenario: Rightmost tab
- **WHEN** the context menu opens for the rightmost tab in its group
- **THEN** "Close Tabs to the Right" is shown disabled

### Requirement: Close actions confirm when a tunnel would disconnect
Any close action (Close Tab, Close Other Tabs, Close Tabs to the Right, Close All Tabs) that would
close the last panel using a context's tunnel SHALL show the existing tunnel-disconnect
confirmation dialog before closing, naming the context(s)/tunnel(s) that would disconnect.

#### Scenario: Closing the last panel on a tunneled context
- **WHEN** the user runs "Close Tab" on the only remaining panel using a context bound to an
  active tunnel
- **THEN** the application shows a confirmation dialog naming the context and tunnel before
  closing the tab

### Requirement: Pinned tabs are excluded from bulk close and sort first
A pinned tab SHALL be excluded from "Close Other Tabs" and "Close All Tabs" in its group, and
pinned tabs SHALL sort before unpinned tabs within their group. Pinned state SHALL persist with
the panel across restarts.

#### Scenario: Bulk close spares pinned tabs
- **WHEN** the user runs "Close All Tabs" in a group containing one pinned tab
- **THEN** every unpinned tab in the group closes and the pinned tab remains open

#### Scenario: Pinned state survives restart
- **WHEN** the user pins a tab and restarts the application
- **THEN** the restored workspace shows that tab pinned in the same group

### Requirement: Move to New Window
"Move to New Window" SHALL remove the tab from its current group and open it as the sole panel in
a new application window, preserving the panel's state (cluster context, resource view, filters,
selection).

#### Scenario: Moving a tab to a new window
- **WHEN** the user runs "Move to New Window" on a tab
- **THEN** the tab closes in its original window and reopens, with its state intact, as the only
  panel in a newly opened window
