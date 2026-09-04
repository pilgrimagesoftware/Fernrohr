## Purpose

A single registry of every user-invokable action, surfaced through a fuzzy command palette and a
user-editable global keymap, so that all functionality is reachable and rebindable from the keyboard.

## ADDED Requirements

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
