# Spec Delta

## MODIFIED Requirements

### Requirement: Kind-specific sections
The object panel SHALL show structured sections for Node, ConfigMap, Secret,
PersistentVolumeClaim, ServiceAccount, ReplicaSet, Deployment, StatefulSet, DaemonSet and Job
objects, and metadata only for any other kind.

#### Scenario: A Node shows its capacity and conditions
- **WHEN** a Node's panel is open
- **THEN** it shows the node's addresses, capacity and allocatable resources, conditions as
  colored badges, node info, and taints

#### Scenario: A ConfigMap shows its data
- **WHEN** a ConfigMap's panel is open
- **THEN** it shows each data key with its value

#### Scenario: A PersistentVolumeClaim links its volume and storage class
- **WHEN** a bound PersistentVolumeClaim's panel is open
- **THEN** it shows the claim's status, capacity and access modes, and its PersistentVolume and
  StorageClass as references

#### Scenario: A workload shows its replicas
- **WHEN** a Deployment's panel is open
- **THEN** it shows desired, ready and available replicas, its selector, and its conditions

#### Scenario: A Node lists the pods scheduled to it
- **WHEN** a Node's panel is open
- **THEN** it shows a Pods field listing, as `resource-links` references, every pod whose
  `spec.nodeName` matches that node

#### Scenario: A node with no pods scheduled
- **WHEN** a Node's panel is open for a node with no pods scheduled to it
- **THEN** the Pods field is shown empty, not omitted

#### Scenario: The pod list can't be retrieved
- **WHEN** the user may read the Node but not list pods cluster-wide
- **THEN** the Node still loads, and the Pods field says why it's empty rather than showing
  nothing or erroring the whole panel
