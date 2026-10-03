# Spec Delta

## Purpose

Finds and queries a connected cluster's Prometheus so the application can show resource usage
over time, and provides the cluster-wide overview panel built on that data.

## ADDED Requirements

### Requirement: Metrics source detection
The application SHALL attempt to locate a connected cluster's Prometheus `Service` by trying
each known provider's label selector (lens, helm, helm-14, operator, stacklight, openshift) in
turn, and SHALL treat a cluster where none match as having no metrics source rather than as an
error.

#### Scenario: A Helm-installed Prometheus is found
- **WHEN** a cluster has a `Service` matching the helm provider's label selector
- **THEN** the application queries through that service and metrics-backed views show data

#### Scenario: No Prometheus installed
- **WHEN** a cluster has no `Service` matching any known provider
- **THEN** every metrics-backed view reports "no metrics source" rather than erroring

### Requirement: Range queries proxy through the apiserver by default
The application SHALL run PromQL range queries through the kube apiserver's service-proxy
endpoint by default, reusing the cluster's existing (possibly tunneled) `kube::Client`, requiring
no additional configuration.

#### Scenario: Querying through a tunneled cluster
- **WHEN** a cluster's connection is bound to an SSH tunnel
- **THEN** metric queries for that cluster succeed over the same tunnel, with no separate metrics
  connection configured

### Requirement: A query failure is distinct from no data
A metrics-backed view SHALL distinguish a query failure (network error, malformed response, a
provider's PromQL template not matching that cluster's metric names) from a successful query that
returns no samples for the requested window, and SHALL report each distinctly to the user.

#### Scenario: A time window with no data
- **WHEN** a pod has existed for less than the selected time window
- **THEN** its graph shows the window with no data before the pod existed, not an error

#### Scenario: A query that fails
- **WHEN** a range query returns an HTTP error or a response the application cannot parse
- **THEN** the view reports that the query failed, distinct from "no data"

### Requirement: Cluster overview panel
The application SHALL provide a dockable overview panel, one per connected cluster context,
showing that cluster's CPU and memory usage graphed over a user-selectable time window (15
minutes, 1 hour, 6 hours, or 24 hours), alongside summary tiles for node count and pod count.

#### Scenario: Opening the overview panel
- **WHEN** the user opens the overview panel for a connected cluster context
- **THEN** it shows CPU and memory usage graphs for that cluster and current node and pod counts

#### Scenario: Changing the time window
- **WHEN** the user selects a different time window in the overview panel
- **THEN** both graphs redraw for the newly selected window

#### Scenario: Reselecting an open overview panel focuses it
- **WHEN** the overview panel for a cluster context is already open and the user requests it again
- **THEN** the existing panel is focused rather than a duplicate opening

### Requirement: The overview panel is fully keyboard-operable
Every overview panel action - opening it, switching time windows - SHALL be reachable from the
keyboard as well as the mouse, following `.claude/rules/keyboard-first.md`: each is a registered
command with a visible key hint.

#### Scenario: Switching time window by keyboard
- **WHEN** the overview panel has focus
- **THEN** each time-window option is reachable and selectable without a mouse, with its key
  shown
