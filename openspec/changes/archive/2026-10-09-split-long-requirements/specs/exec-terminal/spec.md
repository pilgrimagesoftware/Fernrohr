# Spec Delta

## MODIFIED Requirements

### Requirement: Terminal input goes to the container

While a shell session panel has focus and its session is running, the application SHALL send
every key press, including control keys (Ctrl-C, Ctrl-D, Ctrl-Z), Tab, Escape, arrow keys and
function keys, to the container as terminal input. It SHALL NOT trigger application commands from
those keys, except for a small reserved set: the platform's copy and paste shortcuts, the command
palette, and the panel focus and tab navigation shortcuts.

#### Scenario: Interrupt

- **WHEN** a long-running command is executing in the session and the user presses Ctrl-C
- **THEN** the interrupt reaches the container and the command stops, and no application command runs

#### Scenario: Letter keys that are app shortcuts elsewhere

- **WHEN** the user types `d`, `y`, `s` or `l` in a running session
- **THEN** the characters reach the container and no Fernrohr command runs

#### Scenario: Tab completion

- **WHEN** the user types a partial path and presses Tab in a shell that supports completion
- **THEN** the shell completes it, and focus does not leave the panel

## ADDED Requirements

### Requirement: The shell panel's hint row lists its reserved keys

The shell session panel's hint row SHALL list the reserved set of keys that keep their
application commands while its session is running.

#### Scenario: Reserved keys are shown

- **WHEN** a shell session panel with a running session is shown
- **THEN** its hint row lists the copy and paste shortcuts, the command palette, and the panel focus and tab navigation shortcuts

### Requirement: Pasting into a shell uses bracketed paste

Text pasted into a running shell session SHALL be sent using bracketed paste when the program
in the container enables it.

#### Scenario: Paste into a program that enables bracketed paste

- **WHEN** the program in the container has enabled bracketed paste and the user pastes text into the session
- **THEN** the text reaches the container wrapped in bracketed-paste markers
