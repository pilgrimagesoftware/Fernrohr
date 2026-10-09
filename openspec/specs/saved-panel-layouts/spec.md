# saved-panel-layouts Specification

## Purpose
Lets a user capture a window's current panel arrangement under a name and bring it back later, as
many times as needed, independent of the single implicit arrangement the application already
restores automatically on relaunch.

## Requirements

### Requirement: Saving a layout
The application SHALL let the user save the current window's arrangement under a name: the dock tree
(splits, panel sizes, tab order, active tab, and zoom state), the Resource panel's width and
visibility, each panel's content key (kind, object, namespace selection, and cluster context), each
list panel's view state (its filter text and its sort column and direction), and the window's size.

#### Scenario: Saving a new layout
- **WHEN** the user runs Save Panel Layout and enters a name that does not already identify a
  saved layout
- **THEN** a new saved layout is created capturing the window's current dock tree, panel content
  keys, Resource panel width and visibility, and window size

#### Scenario: Namespaces and filters are saved and restored
- **WHEN** a list panel is scoped to one or more namespaces, has filter text entered (where the
  panel has a filter), and is sorted by a column, and the user saves a layout and later loads it
- **THEN** the restored panel is scoped to the same namespaces, shows the same filter text with
  its rows filtered by it, and is sorted by the same column in the same direction

#### Scenario: Saving over an existing name
- **WHEN** the user enters a name that already identifies a saved layout, including one that
  differs from it only in case
- **THEN** the application asks for confirmation before replacing the existing saved layout's
  contents

#### Scenario: A revealed Secret value is never part of a saved layout
- **WHEN** a panel in the window has a Secret value revealed and the user saves a layout
- **THEN** the saved layout contains no Secret value, consistent with `object-detail`'s existing
  guarantee that a revealed value cannot be serialized

### Requirement: Listing and managing saved layouts
The application SHALL provide a picker listing every saved layout by name, from which the user can
load, rename, or delete a saved layout.

#### Scenario: Opening the picker with no saved layouts
- **WHEN** the user opens the saved layouts picker and no layout has been saved yet
- **THEN** the picker shows that there are no saved layouts, rather than an empty list with no
  explanation

#### Scenario: Renaming a saved layout
- **WHEN** the user renames a saved layout to a name that does not identify another saved layout
- **THEN** the saved layout is listed under the new name and its contents are unchanged

#### Scenario: Renaming to a name already in use
- **WHEN** the user renames a saved layout to a name that already identifies a different saved
  layout, including one that differs from it only in case
- **THEN** the rename is rejected and both saved layouts keep their original names

### Requirement: A Settings section lists and removes saved layouts
The application's Settings window SHALL have a "Layouts" section, reachable and keyboard-operable
the same way its other sections are, that lists every saved layout by name and lets the user remove
one.

#### Scenario: Layouts section lists saved layouts
- **WHEN** the user shows the Layouts section in Settings and two layouts are saved
- **THEN** both are listed by name

#### Scenario: Removing a saved layout asks for confirmation
- **WHEN** the user removes a saved layout from the Layouts section
- **THEN** a confirmation opens with its Cancel control focused, and the layout is not removed
  until the user confirms deliberately

#### Scenario: Enter cancels a removal confirmation
- **WHEN** the removal confirmation is open and the user presses Enter without otherwise acting
- **THEN** the confirmation closes and the saved layout is not removed

#### Scenario: Confirming removes the saved layout
- **WHEN** the user confirms the removal
- **THEN** the saved layout no longer appears in the Layouts section or in the saved layouts picker

#### Scenario: The Layouts section is keyboard-operable
- **WHEN** the user operates the Settings window using only the keyboard
- **THEN** the user can show the Layouts section, move through its list, and remove a saved layout,
  including confirming or cancelling the removal, without using the mouse

### Requirement: Loading a saved layout
The application SHALL let the user load a saved layout into the current window in one of two ways,
as two distinct actions: Add, which opens the saved layout's panels alongside the panels already
open in the window, and Replace, which replaces the window's current arrangement with the saved one
entirely, including its Resource panel state and window size.

#### Scenario: Loading with Replace
- **WHEN** the user loads a saved layout with Replace
- **THEN** the window's arrangement - its dock tree, panels, Resource panel state, and window
  size - is replaced by the saved layout's

#### Scenario: Loading with Add
- **WHEN** the user loads a saved layout with Add while the window already shows other panels
- **THEN** the saved layout's panels open alongside the window's existing panels, and the window's
  Resource panel state and window size are unchanged

#### Scenario: Loading with Add does not duplicate an already-open panel
- **WHEN** the user loads with Add a saved layout containing a panel with the same content key
  (kind, object, namespace, and cluster context) as a panel already open in the window
- **THEN** that panel is not duplicated; the window still shows it once

### Requirement: Restoring never silently drops state
Loading a saved layout, by either Add or Replace, SHALL NOT silently omit a panel. A saved panel
whose cluster context is not currently held by the window - whether or not that context exists in
the kubeconfig, or is connected elsewhere - SHALL be restored as a placeholder stating that the
window is not connected to that context, in the same position in the arrangement, rather than being
dropped, causing an error, or connecting that context without the user asking to.

#### Scenario: Saved context not held by the window
- **WHEN** a saved layout is loaded and one of its panels is scoped to a cluster context the window
  does not currently hold
- **THEN** that panel is restored as a placeholder stating the window is not connected to that
  context, and every other panel in the layout restores normally

#### Scenario: Saved namespace or object no longer exists
- **WHEN** a saved layout references a namespace or object that no longer exists in an otherwise
  reachable cluster context
- **THEN** that panel is restored as a placeholder stating the namespace or object no longer
  exists, and every other panel in the layout restores normally

#### Scenario: An unrecognized panel kind does not fail the whole load
- **WHEN** a saved layout's dock data names a panel kind this build does not recognize
- **THEN** that panel is restored as a placeholder stating it was not restored, and every other
  panel in the layout restores normally

### Requirement: Saved layouts are stored one file per layout
The application SHALL persist each saved layout as its own file, in a versioned format, under a
`layouts/` folder in the application's state directory, created the first time a layout is saved.

#### Scenario: Saved layouts survive a restart
- **WHEN** the application is restarted after a layout was saved
- **THEN** that saved layout still appears in the saved layouts picker and in the Settings Layouts
  section

#### Scenario: One unreadable file does not affect the others
- **WHEN** the `layouts/` folder contains one file that fails to parse and two that parse
  successfully
- **THEN** the two valid saved layouts are listed normally, the unreadable file is reported by its
  filename as unreadable, and the unreadable file is left unchanged on disk

#### Scenario: The layouts folder is created on first save
- **WHEN** the user saves the first layout and no `layouts/` folder exists yet
- **THEN** the folder is created and the layout is saved inside it

### Requirement: Saving, loading, and managing saved layouts are keyboard-operable commands
Save Panel Layout and the saved-layouts picker SHALL each be a registered command with a stable
identifier, a title, a default key binding, and a Window-menu entry, reachable from the command
palette. Within the picker, loading with Add, loading with Replace, renaming, and deleting SHALL
each be reachable by keyboard, and moving the keyboard highlight SHALL be unaffected by mouse
hover.

#### Scenario: Save and the picker are registered commands
- **WHEN** the user opens the command palette
- **THEN** Save Panel Layout and the saved-layouts picker command both appear, each showing its
  current key binding

#### Scenario: Hover does not move the keyboard selection
- **WHEN** the user is navigating the saved layouts picker by keyboard and the mouse passes over a
  different row
- **THEN** the keyboard-selected row does not change

#### Scenario: The picker is fully operable by keyboard
- **WHEN** the user opens the saved layouts picker using only the keyboard
- **THEN** the user can move the selection, load it with Add or with Replace, rename it, and
  delete it (including confirming or cancelling the deletion) without using the mouse

### Requirement: Saving over an existing layout name is confirmed
Saving a layout under a name that already identifies a saved layout (compared without regard to
case) SHALL ask for confirmation before overwriting it.

#### Scenario: Saving over a name that differs only in case
- **WHEN** the user saves a layout under a name that differs only in case from an existing saved
  layout's
- **THEN** the application asks for confirmation before replacing that saved layout's contents

### Requirement: Removing a saved layout is confirmed as irreversible
Removing a saved layout SHALL ask for confirmation as an irreversible action: the confirmation SHALL
open with focus on its Cancel control, so that pressing Enter cancels rather than confirms, and
removing SHALL happen only on a deliberate action - clicking the destructive control, reaching and
activating it by Tab, or its dedicated shortcut.

#### Scenario: Removal waits for a deliberate confirmation
- **WHEN** the user removes a saved layout and the confirmation opens
- **THEN** its Cancel control is focused, and the layout is removed only when the user clicks the
  destructive control, activates it after reaching it by Tab, or presses its dedicated shortcut

### Requirement: Loading a layout keeps the current window
Neither way of loading a saved layout SHALL open a new window, and neither SHALL move the current
one: an open window's position can't be set by the application.

#### Scenario: Loading stays in the current window
- **WHEN** the user loads a saved layout with Add or with Replace
- **THEN** the layout loads into the current window, no new window opens, and the window's position
  is unchanged

### Requirement: Missing or unrecognized panels restore as placeholders
When a saved layout is loaded, a panel whose namespace or object no longer exists, or whose panel
kind this build does not recognize, SHALL be restored as a placeholder stating the problem rather
than being dropped or causing an error.

#### Scenario: A deleted object restores as a placeholder
- **WHEN** a saved layout is loaded and one of its panels shows an object that no longer exists
- **THEN** that panel is restored as a placeholder stating the object no longer exists, and the load
  does not fail

### Requirement: A saved layout's display name is stored in its file
A saved layout's filename SHALL be derived from its display name, and the display name itself SHALL
be stored inside the file, independent of the derived filename.

#### Scenario: The display name comes from the file
- **WHEN** the user saves a layout and the application is later restarted
- **THEN** the layout is listed under the display name stored in its file

### Requirement: An unreadable saved layout file is skipped
A single saved-layout file that cannot be parsed SHALL be skipped and reported as unreadable, by its
filename, without affecting any other saved layout, and SHALL be left on disk unmodified.

#### Scenario: An unparseable file beside valid ones
- **WHEN** the `layouts/` folder contains a file that fails to parse beside files that parse
- **THEN** the valid saved layouts are listed normally, and the unreadable file is reported by its
  filename and left unchanged on disk

### Requirement: Saved layout files are never partially written
A save or rename of a saved layout SHALL be written so that an interruption leaves either the
previous saved content or the new saved content on disk, never a partially written file.

#### Scenario: An interrupted save
- **WHEN** a save of an existing saved layout is interrupted before it completes
- **THEN** that layout's file holds either its previous content or its new content, never a partial
  write
