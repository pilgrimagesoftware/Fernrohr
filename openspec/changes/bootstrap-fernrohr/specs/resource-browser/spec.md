## Purpose

A panel that shows a live, continuously updated table of a Kubernetes resource kind (Pods in this
change), scoped by namespace and refined by filtering and sorting, backed by a shared watch stream.

## ADDED Requirements

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
resource name, and SHALL let the user sort by any displayed column.

#### Scenario: Text filter

- **WHEN** the user types `nginx` into the filter
- **THEN** only rows whose Pod name contains `nginx` remain visible
- **AND** clearing the filter restores all rows

#### Scenario: Column sort

- **WHEN** the user clicks the Age column header
- **THEN** rows sort by age, and clicking again reverses the order

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
