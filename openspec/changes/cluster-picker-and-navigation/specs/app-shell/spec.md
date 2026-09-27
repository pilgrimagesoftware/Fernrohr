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

#### Scenario: Saved layout is restored per cluster

- **WHEN** a window connects to a cluster it has a saved layout for
- **THEN** that layout is restored instead of showing the full Resource panel

#### Scenario: No saved layout for this cluster shows the Resource panel

- **WHEN** a window connects to a cluster it has no saved layout for
- **THEN** the workspace shows the Resource panel listing that cluster's discovered
  resources (see `resource-browser`) instead of restoring an unrelated layout

## ADDED Requirements

### Requirement: Resource panel placement

The Resource panel SHALL be anchored to either edge of the main window, defaulting to a
user preference, movable to the other edge at runtime, and collapsible to reclaim window
space.

#### Scenario: Default anchor follows preference

- **WHEN** a window opens with a workspace
- **THEN** the Resource panel appears on the edge set by the user's preference

#### Scenario: Moving the Resource panel at runtime

- **WHEN** the user moves the Resource panel to the opposite edge
- **THEN** it re-anchors there for that window without affecting the user's saved
  preference unless the user also updates that preference

#### Scenario: Collapsing the Resource panel

- **WHEN** the user collapses the Resource panel
- **THEN** the workspace reclaims that space and other panels are unaffected

### Requirement: Panel focus

Exactly one dockable panel in a workspace SHALL hold focus at a time, be visually
distinguished as focused, and receive panel-scoped keyboard shortcuts.

#### Scenario: Clicking a panel focuses it

- **WHEN** the user clicks a dockable panel that does not have focus
- **THEN** that panel becomes focused, is visually distinguished from unfocused panels,
  and the previously focused panel loses that distinction

#### Scenario: Keyboard shortcuts target the focused panel

- **WHEN** a panel-scoped keyboard shortcut is invoked
- **THEN** it is dispatched to the currently focused panel, not any other open panel

### Requirement: Panel maximize

A dockable panel other than the Resource panel SHALL be maximizable to fill the
workspace area, excluding the Resource panel's space, with at most one panel maximized
per window at a time.

#### Scenario: Maximizing a panel

- **WHEN** the user maximizes a dockable panel
- **THEN** that panel fills the workspace area other than the Resource panel, and any
  other open panels are hidden until it is restored

#### Scenario: Maximizing a second panel restores the first

- **WHEN** the user maximizes a panel while another panel is already maximized
- **THEN** the previously maximized panel returns to its prior position and the newly
  maximized panel fills the workspace instead
