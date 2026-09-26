## Purpose

The application window frame: a docked, resizable panel workspace that can span multiple OS windows,
with window geometry and open panels restored on the next launch.

## ADDED Requirements

### Requirement: Main window with docked panel workspace

The application SHALL present a main window containing a panel workspace where panels can be arranged
by docking and splitting, and resized by dragging their separators.

#### Scenario: First launch shows an empty workspace

- **WHEN** the application starts with no saved workspace state
- **THEN** a single main window opens with an empty panel area and a visible way to add a panel

#### Scenario: Panels can be split and resized

- **WHEN** the user adds a second panel to a workspace that already has one
- **THEN** the workspace shows both panels in a split arrangement
- **AND** dragging the separator between them changes their relative sizes

#### Scenario: Panels can be closed

- **WHEN** the user closes a panel
- **THEN** the panel is removed and the remaining panels reflow to fill the space

### Requirement: Multiple windows

The application SHALL allow the user to open additional windows, each with an independent panel
workspace, and SHALL keep running while at least one window is open.

#### Scenario: Open a new window

- **WHEN** the user invokes the "new window" action
- **THEN** a second window opens with its own empty panel workspace
- **AND** panels in one window are unaffected by changes in the other

#### Scenario: Closing the last window exits

- **WHEN** the user closes the only remaining window
- **THEN** the application exits cleanly, persisting current state first

### Requirement: Workspace persistence

The application SHALL persist, per window, the window's screen geometry and the set, arrangement, and
configuration of its open panels, and SHALL restore them on the next launch.

#### Scenario: Layout restored on relaunch

- **WHEN** the user has two windows with several arranged panels and quits the application
- **AND** the application is launched again
- **THEN** both windows reopen at their previous size and position with the same panels arranged the same way

#### Scenario: Corrupt or unreadable state falls back to defaults

- **WHEN** the saved workspace state file cannot be parsed
- **THEN** the application starts with a single default main window
- **AND** the unreadable file is left in place rather than overwritten
