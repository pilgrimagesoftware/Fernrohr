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
Closing the last open panel in a window SHALL close it like any other, and the window SHALL then
return to the cluster picker.

#### Scenario: Closing the only panel
- **WHEN** a window has exactly one open panel and the user clicks its close control or presses
  `Cmd-W`
- **THEN** the panel closes and the window shows the cluster picker
