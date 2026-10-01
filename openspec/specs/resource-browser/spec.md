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

### Requirement: Resource kinds are grouped into collapsible sections

The Resource panel SHALL present every discovered kind under exactly one named
section, rather than as one flat list, and SHALL show no kind twice. No kind may
be dropped by grouping: the union of the sections is exactly the set of kinds the
cluster's API discovery reported.

A kind's section SHALL be determined by its API group **and** its resource
plural together. A section SHALL NOT be inferred from the API group alone.

#### Scenario: Grouping loses no kind

- **WHEN** a cluster's discovery reports a kind that matches no known section
- **THEN** that kind appears under Custom Resources
- **AND** the total count of rows across all sections equals the number of
  discovered kinds

#### Scenario: Opening a kind still works from a section

- **WHEN** the user opens a kind from within a section
- **THEN** the same panel opens as it would from a flat list

### Requirement: Resource kinds carry a category

Every discovered kind SHALL be assigned exactly one category from a fixed set -
Workloads, Config, Network, Storage, Cluster, Access Control, or Custom
Resources - and sections SHALL be presented in that fixed order, not
alphabetically, with Custom Resources last.

#### Scenario: Core-group kinds land in different sections

- **WHEN** a cluster reports `Pod`, `Service`, `ConfigMap`,
  `PersistentVolumeClaim`, `ServiceAccount` and `Namespace`, all of which are in
  the core API group
- **THEN** they appear under Workloads, Network, Config, Storage, Access
  Control and Cluster respectively

#### Scenario: A CRD lands in Custom Resources

- **WHEN** a cluster defines a CRD that is not one of the built-in Kubernetes
  kinds
- **THEN** it appears under Custom Resources

#### Scenario: Sections appear in a fixed order

- **WHEN** several sections are non-empty
- **THEN** they are ordered Workloads, Config, Network, Storage, Cluster, Access
  Control, Custom Resources

#### Scenario: An empty section takes no space

- **WHEN** a section holds no kinds for this cluster
- **THEN** it is not rendered at all

### Requirement: Sections are collapsible

Each rendered section SHALL have a header that collapses and expands its rows,
and SHALL show how many kinds it holds. Collapse state SHALL be held per window
and SHALL NOT be written to the user's preference file, so a new window opens
with every section expanded.

#### Scenario: Collapsing a section hides its rows

- **WHEN** the user collapses a section
- **THEN** that section's rows are hidden
- **AND** its header remains, still showing the section's name and kind count

#### Scenario: A new window is not collapsed

- **WHEN** a user collapsed sections in one window and then opens a new window
- **THEN** the new window's sections are all expanded

### Requirement: Filtering the Resource panel

The Resource panel SHALL provide a filter box pinned to its bottom edge that
narrows the visible rows across every section at once. Matching SHALL be a
case-insensitive substring test against a row's label, kind, resource plural and
API group.

While a filter is active, a section holding at least one matching row SHALL be
rendered expanded regardless of its collapse state, and a section holding no
matching row SHALL be hidden. Filtering SHALL NOT change the stored collapse
state, so clearing the filter restores the panel to the shape the user left it
in.

#### Scenario: A filter narrows every section at once

- **WHEN** the user types a substring that matches kinds in more than one
  section
- **THEN** only matching rows are shown, in each of those sections

#### Scenario: Matches are visible even in a collapsed section

- **WHEN** a section is collapsed and the filter matches a row inside it
- **THEN** that section is shown expanded with the matching row visible

#### Scenario: Clearing the filter restores the previous shape

- **WHEN** the user clears the filter
- **THEN** every section returns to the collapse state it had before the filter
  was typed

#### Scenario: A filter matching nothing says so

- **WHEN** the filter matches no kind in the cluster
- **THEN** the panel shows no sections and reports that nothing matched

#### Scenario: Group names are searchable

- **WHEN** the user filters on an API group name such as `batch`
- **THEN** kinds in that group match, even where the group name is in neither
  the row's label nor its kind

### Requirement: The Resource panel is fully keyboard-operable

Every Resource panel action SHALL be reachable from the keyboard as well as the mouse
(`.claude/rules/keyboard-first.md`): moving between kinds, opening one, collapsing and
expanding a section, and focusing, typing into and clearing the filter. One selection
SHALL be moved by both clicks and the keyboard; hover SHALL NOT move it. The panel SHALL
show its keys in a hint row.

#### Scenario: Navigate and open from the keyboard

- **WHEN** the Resource panel has focus and the user presses Down twice, then Enter
- **THEN** the second visible kind below the current selection opens, as a double-click would open it

#### Scenario: Collapse and expand a section from the keyboard

- **WHEN** a kind in the Workloads section is selected and the user presses Left
- **THEN** Workloads collapses, and Right expands it again

#### Scenario: Filter from the keyboard

- **WHEN** the Resource panel has focus and the user presses `/`, types `ingr` and presses Escape
- **THEN** the filter field takes focus and narrows the list to matching kinds, and Escape clears it and returns focus to the list
