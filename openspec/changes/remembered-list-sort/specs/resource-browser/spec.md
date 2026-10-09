# Spec Delta

## MODIFIED Requirements

### Requirement: Filtering and sorting

A resource-browser panel SHALL provide a text filter that narrows rows by substring match on the
resource name, and SHALL let the user sort by any displayed column, cycling ascending,
descending, and back to the default sort, with the current sort column and direction shown by an
indicator on that column's header. The default sort SHALL be the kind's first column in its
default column order, ascending. A table SHALL always be sorted.

#### Scenario: Text filter

- **WHEN** the user types `nginx` into the filter
- **THEN** only rows whose Pod name contains `nginx` remain visible
- **AND** clearing the filter restores all rows

#### Scenario: Clicking a header sorts ascending

- **WHEN** the user clicks an unsorted column's header
- **THEN** the table's rows sort by that column ascending, and that header shows an ascending
  indicator

#### Scenario: Clicking again reverses direction

- **WHEN** the user clicks an already-ascending-sorted column's header
- **THEN** the table's rows sort by that column descending, and that header shows a descending
  indicator

#### Scenario: Third click returns to the default sort

- **WHEN** the user clicks an already-descending-sorted Age column's header in a Pods table
- **THEN** the rows sort by Name ascending, and the Name header shows an ascending indicator

#### Scenario: Default sort with reordered columns

- **WHEN** the user has dragged the Status column ahead of Name and a sort returns to the default
- **THEN** the rows sort by Name ascending, because Name is first in the kind's default column order

## ADDED Requirements

### Requirement: Remembered sort per resource kind

The application SHALL remember the last sort the user chose in a list panel for each resource kind, across contexts, windows, and restarts. A newly opened list panel of that kind with no sort of its own SHALL start with the remembered sort, or with the default sort when none is remembered or the remembered column no longer exists. A panel restored with its own sort SHALL keep it.

#### Scenario: New panel uses the remembered sort

- **WHEN** the user sorts a Deployments panel by Age descending and later opens a Deployments panel for another context
- **THEN** the new panel opens sorted by Age descending

#### Scenario: First panel of a kind

- **WHEN** the user opens a ConfigMaps panel and has never sorted one
- **THEN** it opens sorted by Name ascending

#### Scenario: Restored panel keeps its own sort

- **WHEN** a Pods panel saved in a layout sorted by Restarts is loaded after the user's remembered Pods sort became Name
- **THEN** that panel is sorted by Restarts

#### Scenario: Remembered column is gone

- **WHEN** a custom resource's remembered sort column no longer appears in its columns
- **THEN** a new panel of that kind opens with the default sort

#### Scenario: Remembered across restarts

- **WHEN** the user sorts a Services panel by Type, quits, relaunches, and opens a new Services panel
- **THEN** it opens sorted by Type ascending

### Requirement: Keyboard sorting

A list panel SHALL let the user sort from the keyboard with registered commands, available from the command palette and rebindable: Sort by Next Column, Sort by Previous Column, and Cycle Sort, which steps the current column through ascending, descending, and the default sort. A keyboard sort change SHALL be remembered like a header click.

#### Scenario: Sort without the mouse

- **WHEN** a Pods table has focus and the user presses the Sort by Next Column key, then the Cycle Sort key
- **THEN** the sort moves to the next column ascending, then to that column descending, and a new Pods panel opens with that sort
