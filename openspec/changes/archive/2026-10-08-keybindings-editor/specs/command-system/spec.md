# Spec Delta

## ADDED Requirements

### Requirement: In-app keybindings editor

The application SHALL provide a Settings window, opened by a registered command, with a Keyboard
Shortcuts section. It SHALL list every registered command with where it applies and its current
key binding, marking any binding that differs from the default. It SHALL let the user record a
new binding for a command, reset it to its default, or remove it. A change SHALL take effect
immediately, without a restart, and SHALL be saved to `keymap.toml` in the same format the user
can edit by hand. Every editor action SHALL be reachable from the keyboard.

#### Scenario: Every command is listed

- **WHEN** the user opens Settings and shows Keyboard Shortcuts
- **THEN** every registered command is listed with its title, where it applies, and its current key
- **AND** a command added after the user's `keymap.toml` was first written is listed with its default key
- **AND** a command with no default key is listed and can be given one

#### Scenario: Search the list

- **WHEN** the user types in the Keyboard Shortcuts filter
- **THEN** only commands whose title, identifier or key matches are shown

#### Scenario: Record a new key

- **WHEN** the user starts recording on a command and presses a key combination
- **THEN** that combination becomes the command's binding and works immediately, without a restart
- **AND** the change is saved to `keymap.toml`
- **AND** key hints, palette entries and menu shortcuts show the new key

#### Scenario: Recording a key that is already bound does not trigger it

- **WHEN** the user records a key combination that is bound to another command (for example, the
  window-close key)
- **THEN** the combination is recorded and the other command does not run

#### Scenario: Cancel recording

- **WHEN** the user is recording and presses Escape
- **THEN** recording stops and the command's binding is unchanged

#### Scenario: A conflict is named before it applies

- **WHEN** the user records a key combination that another command already uses where both are
  available (both global, or in the same panel)
- **THEN** the editor names that other command and asks whether to apply anyway
- **AND** declining leaves both bindings unchanged

#### Scenario: A panel key that shadows a global key is shown

- **WHEN** a panel command and a global command share a key combination
- **THEN** the editor shows that inside that panel the panel command takes the key

#### Scenario: Reset and remove

- **WHEN** the user resets a changed command
- **THEN** it returns to its default binding, immediately and in `keymap.toml`
- **AND WHEN** the user removes a command's binding
- **THEN** the command has no key until one is recorded, and it stays available in the command palette

#### Scenario: The editor works from the keyboard

- **WHEN** the Keyboard Shortcuts list has focus
- **THEN** the user can move between rows, start recording, filter, reset and remove without the mouse
