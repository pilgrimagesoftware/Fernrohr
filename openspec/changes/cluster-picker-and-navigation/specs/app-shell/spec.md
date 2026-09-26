# Spec Delta

## MODIFIED Requirements

### Requirement: Main window with docked panel workspace

The application SHALL present a main window containing a panel workspace where panels can be arranged
by docking and splitting, and resized by dragging their separators.

#### Scenario: First launch shows an empty workspace

- **WHEN** the application starts with no saved workspace state
- **THEN** a single main window opens showing the cluster picker (see `cluster-picker`)
  rather than an empty panel area

#### Scenario: Panels can be split and resized

- **WHEN** the user adds a second panel to a workspace that already has one
- **THEN** the workspace shows both panels in a split arrangement
- **AND** dragging the separator between them changes their relative sizes

#### Scenario: Panels can be closed

- **WHEN** the user closes a panel
- **THEN** the panel is removed and the remaining panels reflow to fill the space

#### Scenario: Closing the last panel returns to the picker

- **WHEN** the user closes a window's last remaining panel
- **THEN** that window shows the cluster picker again instead of an empty panel area
