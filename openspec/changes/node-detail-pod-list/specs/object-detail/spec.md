# Spec Delta

## MODIFIED Requirements

### Requirement: Kind-specific sections
The object panel SHALL show structured sections for Node, ConfigMap, Secret,
PersistentVolumeClaim, ServiceAccount, ReplicaSet, Deployment, StatefulSet, DaemonSet, Job,
CronJob, Service, Ingress, Endpoints, EndpointSlice, NetworkPolicy, Namespace, PersistentVolume,
StorageClass, Role, ClusterRole, RoleBinding, ClusterRoleBinding and Event objects, and metadata
only for any other kind.

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

#### Scenario: A CronJob shows its schedule and jobs
- **WHEN** a CronJob's panel is open
- **THEN** it shows its schedule, suspend flag, concurrency policy, last schedule and last
  successful time, and each active Job as a reference

#### Scenario: A Service shows its ports and selector
- **WHEN** a Service's panel is open
- **THEN** it shows its type, cluster IPs, external IPs or load-balancer ingress, each port with
  its protocol, target port and node port, its selector, and its session affinity

#### Scenario: An Ingress links its backends
- **WHEN** an Ingress's panel is open
- **THEN** it shows its class, each rule's host and paths, its TLS hosts, and each backend
  Service and TLS Secret as a reference

#### Scenario: Endpoints link their targets
- **WHEN** an Endpoints or EndpointSlice panel is open
- **THEN** it shows each address with its ready state and ports, and each target Pod as a
  reference

#### Scenario: A NetworkPolicy shows its rules
- **WHEN** a NetworkPolicy's panel is open
- **THEN** it shows its pod selector, policy types, and each ingress and egress rule's peers and
  ports

#### Scenario: A Namespace shows its phase
- **WHEN** a Namespace's panel is open
- **THEN** it shows its phase and conditions

#### Scenario: A PersistentVolume links its claim and storage class
- **WHEN** a bound PersistentVolume's panel is open
- **THEN** it shows its capacity, access modes, reclaim policy, phase and volume source, and its
  claim and StorageClass as references

#### Scenario: A StorageClass shows its provisioner
- **WHEN** a StorageClass's panel is open
- **THEN** it shows its provisioner, parameters, reclaim policy, volume binding mode, whether
  expansion is allowed, and whether it is the cluster default

#### Scenario: A role shows its rules
- **WHEN** a Role's or ClusterRole's panel is open
- **THEN** it shows each rule's API groups, resources, resource names and verbs

#### Scenario: A binding links its role and subjects
- **WHEN** a RoleBinding's or ClusterRoleBinding's panel is open
- **THEN** it shows its role and each subject, with the role and each ServiceAccount subject as
  references

#### Scenario: An Event shows what happened and to what
- **WHEN** an Event's panel is open
- **THEN** it shows its type as a colored badge, reason, full message, count, first and last seen
  times, reporting component and instance, action, and its involved object and any related object as
  references, reading `events.k8s.io/v1` fields where the legacy ones are empty

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
