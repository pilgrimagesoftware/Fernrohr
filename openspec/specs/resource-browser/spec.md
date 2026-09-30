## Purpose

A panel that shows a live, continuously updated table of a Kubernetes resource kind (Pods in this
change), scoped by namespace and refined by filtering and sorting, backed by a shared watch stream.

## Requirements

### Requirement: Live Pods table

A resource-browser panel SHALL display Pods for its connected cluster in a table and SHALL reflect
create, update, and delete events from the cluster without the user refreshing.

#### Scenario: Initial population

- **WHEN** a Pods panel opens against a connected cluster
- **THEN** the table lists the current Pods with at least name, namespace, ready count, status, restarts, and age

#### Scenario: Live create and delete

- **WHEN** a Pod is created in the cluster while the panel is open
- **THEN** a row for that Pod appears within a few seconds
- **AND** when that Pod is deleted, its row disappears

#### Scenario: Live status change

- **WHEN** an existing Pod's phase changes from Pending to Running
- **THEN** that row's status cell updates in place without reordering unrelated rows

### Requirement: Namespace scoping

A resource-browser panel SHALL let the user view a single namespace or all namespaces, and SHALL
persist the selection with the panel.

#### Scenario: Restrict to one namespace

- **WHEN** the user selects namespace `kube-system`
- **THEN** the table shows only Pods in `kube-system`

#### Scenario: All namespaces

- **WHEN** the user selects "all namespaces"
- **THEN** the table shows Pods from every namespace the user can list

### Requirement: Filtering and sorting

A resource-browser panel SHALL provide a text filter that narrows rows by substring match on the
resource name, and SHALL let the user sort by any displayed column, cycling ascending,
descending, and unsorted, with the current sort column and direction shown by an indicator on
that column's header.

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

### Requirement: List panel titles use the kind's plural form

A resource-browser panel showing a list of a kind SHALL title its tab and title bar with that
kind's plural display name, not its singular Kubernetes Kind name.

#### Scenario: Pods list panel is titled "Pods"

- **WHEN** a Pods list panel is open
- **THEN** its tab and title bar read "Pods", not "Pod"

### Requirement: Shared, reference-counted watches

The application SHALL maintain at most one watch stream per (cluster, resource kind) regardless of how
many panels display it, and SHALL stop that watch when the last panel using it closes.

#### Scenario: Two panels share one watch

- **WHEN** two Pods panels are open against the same cluster
- **THEN** only one Pod watch stream exists for that cluster

#### Scenario: Watch stops when unused

- **WHEN** the last panel displaying Pods for a cluster closes
- **THEN** the Pod watch stream for that cluster is torn down

#### Scenario: Watch reconnects after a drop

- **WHEN** an active Pod watch stream is interrupted by a network error
- **THEN** the application re-establishes the watch and the table converges to the cluster's current state
