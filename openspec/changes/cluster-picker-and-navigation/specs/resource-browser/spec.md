# Spec Delta

## ADDED Requirements

### Requirement: Resource panel lists all discovered kinds by default

When a window has no saved layout for its connected cluster, the Resource panel SHALL
list every resource kind the cluster's API discovery reports, including CRDs, instead of
a fixed subset.

#### Scenario: Fresh connection shows full discovery

- **WHEN** a window connects to a cluster and has no saved layout for it
- **THEN** the Resource panel lists all resource kinds from that cluster's API discovery,
  including any CRDs it defines

#### Scenario: Cluster dropdown appears with more than one connection

- **WHEN** a window has more than one cluster connection open
- **THEN** the Resource panel shows a dropdown to choose which connection's resources
  the list reflects

#### Scenario: Cluster dropdown absent with one connection

- **WHEN** a window has exactly one cluster connection open
- **THEN** the Resource panel shows no cluster dropdown

### Requirement: Opening a resource kind opens a dockable panel

Selecting a resource kind in the Resource panel, by double-click or from its context
menu, SHALL open a dockable panel for that kind if one is not already open for the same
kind, cluster, and namespace scope.

#### Scenario: Double-click opens a panel

- **WHEN** the user double-clicks a resource kind in the Resource panel
- **THEN** a dockable panel for that kind opens in the workspace

#### Scenario: Context menu opens a panel

- **WHEN** the user selects a resource kind's context menu and chooses to open it
- **THEN** a dockable panel for that kind opens in the workspace, equivalently to
  double-click

### Requirement: Panel title bar identifies kind, cluster, and namespace

A resource panel's title bar SHALL show the resource kind, the cluster name when the
window has more than one connection open, a namespace picker when the kind is
namespaced, a controls menu, and a close button.

#### Scenario: Single cluster omits the cluster name

- **WHEN** a resource panel belongs to a window with exactly one cluster connection
- **THEN** its title bar shows the resource kind without a cluster name

#### Scenario: Multiple clusters show the cluster name

- **WHEN** a resource panel belongs to a window with more than one cluster connection
- **THEN** its title bar shows both the resource kind and that panel's cluster name

#### Scenario: Namespaced kind shows a namespace picker

- **WHEN** a resource panel shows a namespaced kind
- **THEN** its title bar includes a namespace picker

#### Scenario: Cluster-scoped kind omits the namespace picker

- **WHEN** a resource panel shows a cluster-scoped kind
- **THEN** its title bar shows no namespace picker

### Requirement: Resource kind navigation

A connected window SHALL provide a way to switch which resource kind its active panel
displays, choosing from the kinds the application currently supports, without closing
and reopening the window's connection.

#### Scenario: Switch from Pods to Logs

- **WHEN** a window is showing a Pods panel for a connected cluster
- **AND** the user navigates to Logs
- **THEN** the panel is replaced by a Logs view for the same connection, without
  reconnecting or losing the cluster session

#### Scenario: Navigation reflects the connected cluster's discovery

- **WHEN** a window is connected to a cluster
- **THEN** the navigation only offers resource kinds the application supports and that
  cluster's API discovery confirms are available
