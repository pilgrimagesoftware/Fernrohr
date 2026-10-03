# Spec Delta

## MODIFIED Requirements

### Requirement: Tab keys follow tab order
The pod detail panel's tab keys SHALL be positional: 1 Overview, 2 Containers, 3 Configuration, 4
Volumes, 5 Events, 6 Managed Fields, 7 Metrics.

#### Scenario: The Configuration tab key
- **WHEN** a pod detail panel has focus and the user presses `3`
- **THEN** the Configuration tab is shown

#### Scenario: The Metrics tab key
- **WHEN** a pod detail panel has focus and the user presses `7`
- **THEN** the Metrics tab is shown

## ADDED Requirements

### Requirement: Metrics tab graphs the pod's resource usage
The pod detail panel's Metrics tab SHALL graph the pod's aggregate CPU, memory, disk, and network
usage over a user-selectable time window (15 minutes, 1 hour, 6 hours, or 24 hours), and SHALL
mark each graph with the pod's aggregate request and limit as reference lines when the pod sets
them for that resource.

#### Scenario: A pod with requests and limits set
- **WHEN** a pod's containers together request 500m CPU and limit 1 CPU, and the Metrics tab is
  open
- **THEN** the CPU graph shows usage over time with reference lines at 500m and 1 core

#### Scenario: A pod with no limit set for a resource
- **WHEN** a pod sets a memory request but no memory limit
- **THEN** the memory graph shows the request reference line and no limit line

#### Scenario: Disk and network have no request/limit concept
- **WHEN** the Metrics tab is open
- **THEN** the disk and network graphs show usage over time with no reference lines, since
  Kubernetes has no request/limit for those resources

### Requirement: Per-container graphs in a multi-container pod
When a pod has more than one container, the Metrics tab SHALL show, in addition to the pod's
aggregate graphs, one set of CPU, memory, disk, and network graphs per container, each marked
with that container's own request and limit reference lines, and each labeled with its
container's name.

#### Scenario: A two-container pod
- **WHEN** the Metrics tab is open for a pod with containers `app` and `sidecar`
- **THEN** it shows the pod's aggregate graphs plus one labeled graph set for `app` and one for
  `sidecar`, each with its own request/limit reference lines

#### Scenario: A single-container pod shows no separate per-container section
- **WHEN** the Metrics tab is open for a pod with exactly one container
- **THEN** no separate per-container graph set is shown beyond the pod's aggregate graphs, since
  they would be identical

### Requirement: The Metrics tab reports when usage can't be shown
The Metrics tab SHALL report "no metrics source" when the cluster has none, and SHALL report a
query failure distinctly from a time window with no data yet, matching `cluster-metrics`'s query
failure handling.

#### Scenario: No metrics source
- **WHEN** the Metrics tab is open against a cluster with no detected metrics source
- **THEN** the tab says no metrics source is available rather than showing empty graphs

#### Scenario: A newly created pod
- **WHEN** the Metrics tab is open for a pod created less than one minute ago, with a 1-hour
  window selected
- **THEN** the graphs show the window with no data before the pod existed, not an error
