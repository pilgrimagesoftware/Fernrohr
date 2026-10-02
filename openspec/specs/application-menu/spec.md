## Purpose

Gives the application a native menu bar so every major action category is discoverable by
browsing, not just by already knowing the command palette or a keybinding exists.

## Requirements

### Requirement: Fixed top-level menu structure

The application SHALL present a native menu bar with exactly seven top-level menus, in this
order: App, Context, Edit, View, Navigate, Window, Help.

#### Scenario: Menu bar present on launch

- **WHEN** the application finishes starting up
- **THEN** the native menu bar shows App, Context, Edit, View, Navigate, Window, and Help, in
  that order, before any window is focused

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

### Requirement: Context menu manages cluster context selection

The Context menu SHALL contain the actions for opening the cluster picker and switching the
active cluster context, in place of a conventional document-oriented File menu.

#### Scenario: Context menu lists context actions

- **WHEN** the user opens the Context menu
- **THEN** it shows the action to open the cluster picker

### Requirement: The Window menu lists every open window by its own title

The Window menu SHALL list each open application window as its own entry, labelled with that
window's title, so that two windows connected to different clusters can be told apart from the
menu. Selecting an entry SHALL bring that window forward.

#### Scenario: Two windows on different clusters are distinguishable

- **WHEN** two windows are open, one connected to `staging` and one to `production`
- **THEN** the Window menu lists both as separate entries, each naming its own cluster

#### Scenario: Selecting a window's entry brings it forward

- **WHEN** the user selects a window's entry in the Window menu
- **THEN** that window becomes the active window

#### Scenario: Closing a window drops its entry

- **WHEN** the user closes a window
- **THEN** the Window menu no longer lists an entry for it
