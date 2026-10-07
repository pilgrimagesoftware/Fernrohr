# Spec Delta

## MODIFIED Requirements

### Requirement: Kind-specific sections
The object panel SHALL show structured sections for Node, ConfigMap, Secret,
PersistentVolumeClaim, ServiceAccount, ReplicaSet, Deployment, StatefulSet, DaemonSet, Job,
CronJob, Service, Ingress, Endpoints, EndpointSlice, NetworkPolicy, Namespace, PersistentVolume,
StorageClass, Role, ClusterRole, RoleBinding, ClusterRoleBinding and Event objects. Any other kind
whose object carries a non-empty `status.conditions` array SHALL show a generic Status section
instead; any other kind with neither a dedicated section nor conditions SHALL show metadata only.

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

#### Scenario: A Kustomization with no dedicated section gets a generic Status section
- **WHEN** a Kustomization's panel is open and its CRD has no kind-specific section defined
- **THEN** it shows a Status section with its derived message and its full conditions list, rather
  than metadata only

#### Scenario: A CRD with no conditions keeps metadata only
- **WHEN** a CRD object's panel is open and it has neither a kind-specific section nor
  `status.conditions`
- **THEN** it shows metadata only, unchanged from today

### Requirement: Values can be copied
The pod detail and object detail panels SHALL offer a Copy Resource Name command (palette and key)
for the panel's object, and a copy control on copyable values - container images, ConfigMap keys and
values, a derived status message, and similar single values - that appears on hover and is
reachable by keyboard. A Secret value SHALL be copyable only while revealed.

#### Scenario: Copying the name
- **WHEN** a detail panel has focus and the user runs Copy Resource Name
- **THEN** the object's name is on the clipboard

#### Scenario: Copying an image
- **WHEN** the user hovers a container's image and clicks its copy control
- **THEN** the full image reference is on the clipboard

#### Scenario: Copying a status message
- **WHEN** the user hovers an object's derived status message and clicks its copy control
- **THEN** the full message text is on the clipboard

## ADDED Requirements

### Requirement: A derived status message appears near the top of the panel

The object panel SHALL show a single derived status message for any object that reports
`status.conditions`, near the top of the panel: the `Ready` condition's message when present and
non-empty, toned Good for `True`, Bad for `False`, and Warning for any other status; otherwise the
most recently transitioned condition's non-empty message, untoned. An object with no conditions, or
no condition with a non-empty message, SHALL show no message field. This applies both to kinds that
already show `status.conditions` as badges and to kinds that gain the generic Status section above -
the message is additional to, not a replacement for, the full conditions list. When the object has
a `Ready` condition, the panel SHALL also show a Ready field with that condition's status (`True`,
`False` or `Unknown`), toned and marked like the list's Ready cell, unless the kind's section already
shows readiness.

#### Scenario: A Deployment shows its message alongside its condition badges
- **WHEN** a Deployment's panel is open and its `Progressing` condition is the most recently
  transitioned with a message, with no `Ready` condition present
- **THEN** the panel shows that message near the top, untoned, in addition to its existing
  Conditions badges

#### Scenario: A failing HelmRelease shows a Bad-toned message
- **WHEN** a HelmRelease's panel is open and its `Ready` condition has `status: "False"` with a
  message describing the failure
- **THEN** the panel shows that message near the top in its Bad tone, alongside the full
  conditions list

#### Scenario: A HelmRelease's Ready status is shown beside its message
- **WHEN** a HelmRelease's panel is open and its `Ready` condition has `status: "False"`
- **THEN** the panel shows a Ready field reading False, with the same readiness indicator and Bad
  tone as the list's Ready cell, next to the message

#### Scenario: An object with no conditions shows no message field
- **WHEN** an object's panel is open and it has no `status.conditions`
- **THEN** the panel shows no message field, unchanged from today
