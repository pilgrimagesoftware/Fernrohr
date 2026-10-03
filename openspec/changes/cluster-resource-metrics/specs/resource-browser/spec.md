# Spec Delta

## ADDED Requirements

### Requirement: Node list shows resource usage columns
The Nodes resource-browser table SHALL show CPU, memory, disk, and pod-count columns sourced from
the cluster's metrics source, each showing current usage against capacity, and SHALL sort like
any other column. When the cluster has no metrics source, these columns SHALL show that usage is
unavailable rather than an error or a blank cell indistinguishable from zero usage.

#### Scenario: Usage against capacity
- **WHEN** the Nodes table is open against a cluster with a detected metrics source
- **THEN** each node's row shows its current CPU and memory usage against that node's capacity,
  its disk size, and the number of pods scheduled to it

#### Scenario: Sorting by a resource column
- **WHEN** the user sorts the Nodes table by the CPU column
- **THEN** rows order by current CPU usage, ascending or descending per the existing sort
  convention

#### Scenario: No metrics source
- **WHEN** the Nodes table is open against a cluster with no detected metrics source
- **THEN** the CPU and memory columns show that usage is unavailable, distinct from a node
  reporting zero usage

#### Scenario: Pod count does not depend on metrics
- **WHEN** the Nodes table is open against a cluster with no detected metrics source
- **THEN** the pod-count column still shows the correct count, since it comes from the pods list
  rather than the metrics source
