# Spec Delta

## ADDED Requirements

### Requirement: Closing a panel closes the panel the user pointed at
Every docked panel tab SHALL carry its own close control that closes that tab's panel. Any close
control drawn for a tab group SHALL close that group's active panel. No close control SHALL close
a panel in a different tab group, whichever panel has focus.

#### Scenario: Closing the unfocused group's panel
- **WHEN** two tab groups are stacked, the upper group's Pods panel has focus, and the user clicks
  the close control of the lower group's RoleBindings tab
- **THEN** RoleBindings closes and Pods stays open with focus

#### Scenario: Closing a background tab
- **WHEN** a tab group holds Pods (active) and Services, and the user clicks the close control on
  the Services tab
- **THEN** Services closes and Pods stays the active tab

### Requirement: The last panel can be closed
Closing the last open panel in a window SHALL close it like any other. The window SHALL stay
connected to its cluster contexts, showing its Resource panel beside an empty panel area that
names the key to open a kind. Keyboard focus SHALL move to the Resource panel, expanding it if it
was collapsed.

#### Scenario: Closing the only panel
- **WHEN** a window has exactly one open panel and the user clicks its close control or presses
  `Cmd-W`
- **THEN** the panel closes, the window stays connected, and the Resource panel has focus

### Requirement: A lone panel keeps its close control beside its title
A tab group holding a single panel SHALL still show that panel's close control immediately beside
its title, where the tab's own close control sits when the group has several, not at the far edge
of the group.

#### Scenario: Closing down to one tab
- **WHEN** a tab group with two panels has one closed, leaving one
- **THEN** the remaining panel's close control is still drawn next to its title
