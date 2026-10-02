# Spec Delta

## MODIFIED Requirements

### Requirement: Context menu manages cluster context selection

The Context menu SHALL contain the actions for opening the cluster picker and switching, adding and
disconnecting cluster contexts, in place of a conventional document-oriented File menu. Every
tunnel action SHALL be grouped together at the bottom of the Context menu, separated from the
context actions above it.

#### Scenario: Context menu lists context actions

- **WHEN** the user opens the Context menu
- **THEN** it shows the action to open the cluster picker

#### Scenario: Tunnel items are grouped last

- **WHEN** the user opens the Context menu
- **THEN** the context actions come first, followed by a separator, followed by every tunnel action
  with no context action among them

## ADDED Requirements

### Requirement: The View menu is grouped
The View menu SHALL group its items by purpose - appearance (theme, text size), the Resource panel
(focus, move, collapse, side preference), panel layout (maximize) and table columns - with a
separator between groups and no item outside a group.

#### Scenario: Grouped with separators
- **WHEN** the user opens the View menu
- **THEN** the theme items sit together, the Resource panel items sit together, and a separator
  divides each group from the next

### Requirement: The Navigate menu holds only global navigation
The Navigate menu SHALL contain only commands that act regardless of which panel has focus: moving
focus to the next or previous panel, focusing the Resource panel, and selecting or cycling tabs.
Commands scoped to one panel type SHALL NOT appear in the menu bar; they SHALL remain in the command
palette, the keymap and that panel's hint row. The menu bar SHALL NOT be rebuilt when focus moves
between panels.

#### Scenario: Pods commands are not in Navigate
- **WHEN** the user opens the Navigate menu
- **THEN** it has no Pods, Logs or list-panel items, and still shows Focus Next Panel, Focus Previous
  Panel, Focus Resources and the tab commands

#### Scenario: Panel commands stay reachable
- **WHEN** the user opens the command palette with a Pods panel focused
- **THEN** "Pods: Describe Selected Pod" is listed and runs as before
