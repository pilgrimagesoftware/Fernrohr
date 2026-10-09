# Spec Delta

## ADDED Requirements

### Requirement: Pending shortcut indicator

While the focused window holds the start of a multi-step key binding and is waiting for its next
key, the application SHALL show the keys typed so far in that window's status bar, followed by an
ellipsis, using the same key notation as the command palette. Alongside them, it SHALL list each
key that would complete a binding available in the current focus context, with that command's
title, ordered as the command palette orders commands. The indicator and list SHALL disappear as
soon as the binding completes, the pending keys are abandoned (a key that completes no binding, the
timeout, or a focus change), or the window loses focus. The indicator SHALL NOT take keyboard
focus, and the next key SHALL go to the binding exactly as it would without the indicator.

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
the Settings panel. It accepts whole seconds from 1 to 10 and defaults to 3. When the keys typed so
far are themselves a complete binding and also the start of a longer one, the application SHALL
wait for the configured time before running the shorter binding. When the keys typed so far are
not a complete binding, the application SHALL keep waiting for the next key with no timeout. A
value outside the range SHALL load as the nearest allowed value without discarding the rest of
`ui.toml`.

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
