# Spec Delta

## ADDED Requirements

### Requirement: Duplicate panels are saved and restored separately
A saved layout SHALL keep each panel's instance number, so a layout holding several instances of
one resource restores each of them, with its own state and number. A layout saved before panels had
instance numbers SHALL restore each panel as the first instance.

#### Scenario: Two instances restored
- **WHEN** a layout holding two instances of a Deployments list, one scoped to `team-a` and one to
  `team-b`, is loaded with Replace
- **THEN** both lists are restored, each with its own scope, and the second's title ends in ` (2)`

#### Scenario: Loading with Add keeps a saved duplicate
- **WHEN** the window shows a Service's detail panel and the user loads with Add a layout holding
  that Service's detail panel as instances 1 and 2
- **THEN** instance 2 opens beside the open panel, and instance 1 is not duplicated

#### Scenario: An older layout
- **WHEN** a layout saved before instance numbers is loaded
- **THEN** every panel restores as it did before, as the first instance

## MODIFIED Requirements

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
  (kind, object, namespace, cluster context, and instance number) as a panel already open in the
  window
- **THEN** that panel is not duplicated; the window still shows it once
