# Spec Delta

## MODIFIED Requirements

### Requirement: Main window with docked panel workspace

The application SHALL present a main window containing a panel workspace where panels can be arranged
by docking and splitting, and resized by dragging their separators. The focused panel group SHALL
also be arrangeable from the keyboard: splitting it in a direction, moving the focused panel into an
adjacent group, closing an entire group at once, and merging a group into an adjacent group.

#### Scenario: First launch shows an empty workspace

- **WHEN** the application starts with no saved workspace state
- **THEN** a single main window opens with an empty panel area and a visible way to add a panel

#### Scenario: Panels can be split and resized

- **WHEN** the user adds a second panel to a workspace that already has one
- **THEN** the workspace shows both panels in a split arrangement
- **AND** dragging the separator between them changes their relative sizes

#### Scenario: Panels can be closed

- **WHEN** the user closes a panel
- **THEN** the panel is removed and the remaining panels reflow to fill the space

#### Scenario: Splitting the focused group by keyboard

- **WHEN** the user invokes a "split group" action with a direction (left, right, up, or down)
  while a panel group has focus
- **THEN** a new split pane opens in that direction holding a copy of the focused panel's
  panel-opening target
- **AND** the new pane receives focus

#### Scenario: Moving the focused panel into an adjacent group by keyboard

- **WHEN** the user invokes a "move panel" action with a direction while a panel has focus and an
  adjacent group exists in that direction
- **THEN** the focused panel is removed from its current group's tab strip and added to the
  adjacent group's tab strip
- **AND** if the panel's original group becomes empty, that group's split pane is removed and the
  remaining panes reflow to fill the space

#### Scenario: Moving a panel with no adjacent group in that direction

- **WHEN** the user invokes a "move panel" action with a direction and no group exists in that
  direction
- **THEN** the workspace is unchanged and no error dialog is shown

#### Scenario: Closing the focused group by keyboard

- **WHEN** the user invokes a "close group" action while a panel group has focus
- **THEN** every panel in that group's tab strip is closed, applying the same confirmation prompt
  used for a single panel to any panel in the group whose state warrants confirming
- **AND** if the user cancels a confirmation, no panel in the group is closed

#### Scenario: Merging the focused group into an adjacent group by keyboard

- **WHEN** the user invokes a "merge group" action with a direction while a panel group has focus
  and an adjacent group exists in that direction
- **THEN** every panel in the focused group's tab strip is added to the adjacent group's tab strip
  in their existing order
- **AND** the focused group's split pane is removed and the remaining panes reflow to fill the
  space

#### Scenario: Merging a group with no adjacent group in that direction

- **WHEN** the user invokes a "merge group" action with a direction and no group exists in that
  direction
- **THEN** the workspace is unchanged and no error dialog is shown
