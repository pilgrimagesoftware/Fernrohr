# Spec Delta

## MODIFIED Requirements

### Requirement: Menu items are sourced from the command registry

Every menu item that corresponds to a user-invokable action SHALL be built from that action's
entry in the command registry, and SHALL invoke the same action the command palette or a keymap
binding would invoke for that command.

#### Scenario: Menu item matches palette entry

- **WHEN** a command is registered with a menu assignment and the user opens that command's menu
- **THEN** the menu shows an item with the command's title and, if one is bound, its current key
  binding
- **THEN** activating the menu item runs the same action as running that command from the palette

#### Scenario: Unassigned commands do not appear in any menu

- **WHEN** a registered command has no menu assignment
- **THEN** no top-level menu shows an item for it, though it SHALL remain available in the
  command palette and keymap

#### Scenario: Report Issue command appears in Help menu

- **WHEN** the user opens the Help menu
- **THEN** it shows the "Report Issue" menu item with its command title and, if bound, its current
  key binding
- **THEN** activating "Report Issue" from the Help menu runs the same action as invoking the
  command from the palette or keyboard
