# Spec Delta

## ADDED Requirements

### Requirement: Columns are resizable and reorderable
A resource-browser table's columns SHALL be resizable by dragging a column boundary and
reorderable by dragging a column header to a new position.

#### Scenario: Resizing a column
- **WHEN** the user drags a column boundary in the Pods table
- **THEN** that column's width changes and neighboring columns adjust accordingly

#### Scenario: Reordering a column
- **WHEN** the user drags a column header to a new position
- **THEN** the table's column order reflects the new position, and each cell still shows the
  value for its own column, not the column now in that visual slot

### Requirement: Columns are sortable with a visible sort indicator
A resource-browser table's columns SHALL be sortable by clicking a column header, cycling
ascending, descending, and unsorted, with the current sort column and direction shown by an
indicator on that column's header.

#### Scenario: Clicking a header sorts ascending
- **WHEN** the user clicks an unsorted column's header
- **THEN** the table's rows sort by that column ascending, and that header shows an ascending
  indicator

#### Scenario: Clicking again reverses direction
- **WHEN** the user clicks an already-ascending-sorted column's header
- **THEN** the table's rows sort by that column descending, and that header shows a descending
  indicator
