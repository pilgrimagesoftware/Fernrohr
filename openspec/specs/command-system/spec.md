## Purpose

A single registry of every user-invokable action, surfaced through a fuzzy command palette and a
user-editable global keymap, so that all functionality is reachable and rebindable from the keyboard.

## Requirements

### Requirement: Central command registry

Every user-invokable action SHALL be registered with a stable identifier, a human-readable title, an
optional default key binding, and a context in which it is available. The palette and the keymap SHALL
derive their entries from this registry.

#### Scenario: Registered action is invokable

- **WHEN** an action is registered with an identifier and handler
- **THEN** it can be invoked by that identifier, appears in the command palette, and can be bound in the keymap

#### Scenario: Context gates availability

- **WHEN** an action declares it is only available when a resource table is focused
- **THEN** invoking it or its key binding while no resource table is focused has no effect and it is not offered in the palette

### Requirement: Fuzzy command palette

The application SHALL provide a command palette that lists available actions by title, filters them by
fuzzy substring match as the user types, shows each action's current key binding, and runs the
selected action.

#### Scenario: Find and run an action

- **WHEN** the user opens the palette and types `new win`
- **THEN** the "New Window" action is shown with its key binding and running it opens a new window

#### Scenario: Unavailable actions are hidden

- **WHEN** the palette is open in a context where an action is unavailable
- **THEN** that action does not appear in the results

### Requirement: User-editable global keymap

The application SHALL load key bindings from a user-editable `keymap.toml` in the preferences
directory that maps key combinations to action identifiers, overriding defaults. On first run the file
SHALL be created containing the effective defaults.

#### Scenario: Override a default binding

- **WHEN** the user maps a key combination to an action identifier in `keymap.toml` and restarts the application
- **THEN** that key combination invokes that action instead of any default bound to it

#### Scenario: First run writes defaults

- **WHEN** the application starts and no `keymap.toml` exists
- **THEN** the file is created listing the default bindings

#### Scenario: Invalid keymap falls back

- **WHEN** `keymap.toml` cannot be parsed or names an unknown action identifier
- **THEN** the application falls back to the default bindings, reports the problem, and leaves the file unchanged

### Requirement: Pending shortcut indicator

While the focused window holds the start of a multi-step key binding and is waiting for its next
key, the application SHALL show the keys typed so far in that window's status bar, followed by an
ellipsis, using the same key notation as the command palette.

#### Scenario: First key of a chord

- **WHEN** a panel group has focus and the user presses `cmd-k`
- **THEN** the status bar shows `⌘K …` and lists the arrow keys and `w` with the titles of the split and close-group commands

#### Scenario: Chord completes

- **WHEN** the indicator shows `⌘K …` and the user presses `left`
- **THEN** the split runs, and the indicator and list disappear

#### Scenario: Chord abandoned

- **WHEN** the indicator shows `⌘K …` and the user presses a key that completes no binding
- **THEN** the indicator and list disappear, and no split, move, merge, or close runs

#### Scenario: Rebound chord shows its new keys

- **WHEN** the user has rebound Split Group Left to `cmd-k h` and presses `cmd-k`
- **THEN** the list shows `h` for Split Group Left, not `left`

#### Scenario: Single-step shortcuts show nothing

- **WHEN** the user presses a shortcut that is a complete single-step binding and the prefix of no other binding
- **THEN** no indicator appears

### Requirement: Adjustable shortcut timeout

The application SHALL provide a "Shortcut timeout" preference, stored in `ui.toml` and editable in
the Settings panel, accepting whole seconds from 1 to 10 and defaulting to 3. When the keys typed so
far are themselves a complete binding and also the start of a longer one, the application SHALL
wait for the configured time before running the shorter binding.

#### Scenario: Default timeout

- **WHEN** a key is bound on its own and also starts a longer binding, the preference is unchanged, and the user presses that key and nothing else
- **THEN** the single-key binding runs about 3 seconds later, and the indicator shows until then

#### Scenario: Changed timeout applies at once

- **WHEN** the user sets Shortcut timeout to 6 seconds in Settings
- **THEN** the next ambiguous pending shortcut waits about 6 seconds, without restarting the application

#### Scenario: Unambiguous prefix never times out

- **WHEN** the user presses `cmd-k`, which is not a binding on its own, and waits longer than the configured timeout
- **THEN** the indicator is still shown, and pressing `w` still runs Close Group

#### Scenario: Out-of-range value in the file

- **WHEN** `ui.toml` sets the shortcut timeout to 30
- **THEN** it loads as 10, and every other preference in the file loads normally

### Requirement: In-app keybindings editor

The application SHALL provide a Settings window, opened by a registered command, with a Keyboard
Shortcuts section. It SHALL list every registered command with where it applies and its current
key binding, marking any binding that differs from the default.

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

### Requirement: Pending shortcut completions are listed

Alongside the pending shortcut indicator, the application SHALL list each key that would complete
a binding available in the current focus context, with that command's title, ordered as the command
palette orders commands.

#### Scenario: Completions for a chord

- **WHEN** a panel group has focus and the user presses `cmd-k`
- **THEN** the status bar lists the arrow keys and `w` with the titles of the split and close-group commands

### Requirement: Pending shortcut indicator clears

The pending shortcut indicator and its list SHALL disappear as soon as the binding completes, the
pending keys are abandoned (a key that completes no binding, the timeout, or a focus change), or the
window loses focus.

#### Scenario: Window loses focus

- **WHEN** the indicator shows `⌘K …` and the window loses focus
- **THEN** the indicator and list disappear

### Requirement: Pending shortcut indicator never takes focus

The pending shortcut indicator SHALL NOT take keyboard focus, and the next key SHALL go to the
binding exactly as it would without the indicator.

#### Scenario: Next key completes the binding

- **WHEN** the indicator shows `⌘K …` and the user presses `left`
- **THEN** the split runs exactly as it would with no indicator shown

### Requirement: An unambiguous shortcut prefix never times out

When the keys typed so far are not a complete binding, the application SHALL keep waiting for the
next key with no timeout.

#### Scenario: Waiting past the timeout

- **WHEN** the user presses `cmd-k`, which is not a binding on its own, and waits longer than the configured timeout
- **THEN** the indicator is still shown, and pressing `w` still runs Close Group

### Requirement: An out-of-range shortcut timeout loads clamped

A Shortcut timeout in `ui.toml` outside 1 to 10 seconds SHALL load as the nearest allowed value
without discarding the rest of `ui.toml`.

#### Scenario: Too large a value

- **WHEN** `ui.toml` sets the shortcut timeout to 30
- **THEN** it loads as 10, and every other preference in the file loads normally

### Requirement: Keybindings can be recorded, reset and removed

The Keyboard Shortcuts section SHALL let the user record a new binding for a command, reset it to
its default, or remove it.

#### Scenario: Record a new key

- **WHEN** the user starts recording on a command and presses a key combination
- **THEN** that combination becomes the command's binding

### Requirement: Keybinding changes apply at once and persist

A change made in the Keyboard Shortcuts section SHALL take effect immediately, without a restart,
and SHALL be saved to `keymap.toml` in the same format the user can edit by hand.

#### Scenario: Reset applies at once

- **WHEN** the user resets a changed command
- **THEN** it returns to its default binding immediately, without a restart, and in `keymap.toml`

### Requirement: The keybindings editor works from the keyboard

Every Keyboard Shortcuts editor action SHALL be reachable from the keyboard.

#### Scenario: Editing without the mouse

- **WHEN** the Keyboard Shortcuts list has focus
- **THEN** the user can move between rows, start recording, filter, reset and remove without the mouse
