# Spec Delta

## Purpose

Lets a user capture a window's current panel arrangement under a name and bring it back later, as
many times as needed, independent of the single implicit arrangement the application already
restores automatically on relaunch.

## ADDED Requirements

### Requirement: Saving a layout
The application SHALL let the user save the current window's arrangement under a name: the dock
tree (splits, panel sizes, tab order, active tab, and zoom state), the Resource panel's width and
visibility, each panel's content key (kind, object, namespace, and cluster context), and the
window's size and position. Saving under a name that already exists SHALL ask for confirmation
before overwriting it.

#### Scenario: Saving a new layout
- **WHEN** the user runs Save Panel Layout and enters a name that does not already exist
- **THEN** a new saved layout is created capturing the window's current dock tree, panel content
  keys, Resource panel width and visibility, and window size and position

#### Scenario: Saving over an existing name
- **WHEN** the user enters a name that already identifies a saved layout
- **THEN** the application asks for confirmation before replacing the existing saved layout's
  contents

#### Scenario: A revealed Secret value is never part of a saved layout
- **WHEN** a panel in the window has a Secret value revealed and the user saves a layout
- **THEN** the saved layout contains no Secret value, consistent with `object-detail`'s existing
  guarantee that a revealed value cannot be serialized

### Requirement: Listing and managing saved layouts
The application SHALL provide a picker listing every saved layout by name, from which the user can
restore, rename, or delete a saved layout.

#### Scenario: Opening the picker with no saved layouts
- **WHEN** the user opens the saved layouts picker and no layout has been saved yet
- **THEN** the picker shows that there are no saved layouts, rather than an empty list with no
  explanation

#### Scenario: Renaming a saved layout
- **WHEN** the user renames a saved layout to a name that does not collide with another saved
  layout
- **THEN** the saved layout is listed under the new name and its contents are unchanged

#### Scenario: Renaming to a name already in use
- **WHEN** the user renames a saved layout to a name that already identifies a different saved
  layout
- **THEN** the rename is rejected and both saved layouts keep their original names

#### Scenario: Deleting a saved layout
- **WHEN** the user deletes a saved layout from the picker
- **THEN** it no longer appears in the picker and restoring it is no longer possible

### Requirement: Restoring a saved layout
The application SHALL let the user restore a saved layout either into the current window,
replacing its arrangement, or into a new window, as two distinct actions. Restoring SHALL
reproduce the saved dock tree, panel content keys, Resource panel width and visibility, and window
size and position.

#### Scenario: Restoring into the current window
- **WHEN** the user restores a saved layout into the current window
- **THEN** that window's arrangement is replaced by the saved layout's dock tree, panels, Resource
  panel state, and window size and position

#### Scenario: Restoring into a new window
- **WHEN** the user restores a saved layout into a new window
- **THEN** a new window opens reproducing the saved layout's dock tree, panels, and Resource panel
  state, sized and positioned from the saved layout

### Requirement: Restoring never silently drops state
Restoring a saved layout SHALL NOT silently omit a panel. A panel whose cluster context is not
currently connected SHALL connect that context on demand. A panel whose cluster context no longer
exists, or whose namespace or object no longer exists, SHALL be restored as a placeholder panel
stating the problem, with an action to reconnect or retry, rather than being dropped or causing an
error.

#### Scenario: Saved context not currently connected
- **WHEN** a saved layout references a cluster context that exists in the kubeconfig but is not
  currently connected
- **THEN** restoring the layout connects that context on demand and the panel shows its content
  once connected

#### Scenario: Saved context no longer exists
- **WHEN** a saved layout references a cluster context that is no longer present in the kubeconfig
- **THEN** that panel is restored as a placeholder stating the context no longer exists, offering
  to pick a different context, and every other panel in the layout restores normally

#### Scenario: Saved namespace or object no longer exists
- **WHEN** a saved layout references a namespace or object that no longer exists in an otherwise
  reachable cluster context
- **THEN** that panel is restored as a placeholder stating the namespace or object no longer
  exists, with an action to retry, and every other panel in the layout restores normally

### Requirement: Saved layouts persist across restarts
The application SHALL persist saved layouts to disk, in a versioned format, so they survive an
application restart. A saved-layouts file that fails to parse SHALL be left on disk untouched and
SHALL be treated as having no saved layouts, rather than causing a startup failure.

#### Scenario: Saved layouts survive a restart
- **WHEN** the application is restarted after a layout was saved
- **THEN** that saved layout still appears in the saved layouts picker

#### Scenario: Unreadable saved-layouts file
- **WHEN** the saved-layouts file on disk cannot be parsed
- **THEN** the application starts with no saved layouts available rather than failing to start,
  and does not overwrite the unreadable file

### Requirement: Saving, managing, and restoring are keyboard-operable commands
Save Panel Layout and the saved-layouts picker SHALL each be a registered command with a stable
identifier, a title, a default key binding, and a Window-menu entry, reachable from the command
palette. Within the picker, restoring (in place and into a new window), renaming, and deleting
SHALL each be reachable by keyboard, and moving the keyboard highlight SHALL be unaffected by mouse
hover.

#### Scenario: Save and manage are registered commands
- **WHEN** the user opens the command palette
- **THEN** Save Panel Layout and the saved-layouts picker command both appear, each showing its
  current key binding

#### Scenario: Hover does not move the keyboard selection
- **WHEN** the user is navigating the saved layouts picker by keyboard and the mouse passes over a
  different row
- **THEN** the keyboard-selected row does not change

#### Scenario: The picker is fully operable by keyboard
- **WHEN** the user opens the saved layouts picker using only the keyboard
- **THEN** the user can move the selection, restore it in place or into a new window, rename it,
  and delete it without using the mouse
