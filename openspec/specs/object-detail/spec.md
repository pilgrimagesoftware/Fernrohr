# object-detail Specification

## Purpose
A dockable panel showing one Kubernetes object of any kind the cluster reports, opened by
following a reference to it: its metadata, the sections that matter for its kind, the events
naming it, and its YAML, without ever showing a Secret's values.

## Requirements

### Requirement: Any discovered kind has a single-object panel
The application SHALL open a detail panel for one object of any kind its cluster's API discovery
reports, scoped to that object's cluster context, kind, namespace (for namespaced kinds) and
name. Requesting an object whose panel is already open in that context SHALL focus that panel.

#### Scenario: Opening an object of a kind with no dedicated viewer
- **WHEN** the user follows a reference to an object of a discovered kind that has no
  kind-specific sections
- **THEN** a panel opens showing that object's creation time, name, namespace, labels,
  annotations and owners

#### Scenario: Same object requested twice
- **WHEN** an object's panel is already open and the user follows another reference to it in the
  same cluster context
- **THEN** the existing panel is focused and no second panel opens

#### Scenario: Restored with the window
- **WHEN** the application restarts with an object's panel in a saved window layout
- **THEN** the panel reopens over the same object in the same cluster context

### Requirement: Owners and other references in the panel are links
The object panel SHALL show its object's namespace, each owner reference, and every reference its
kind-specific sections name, as `resource-links` references.

#### Scenario: A ReplicaSet links to its Deployment
- **WHEN** a ReplicaSet's panel shows a ReplicaSet owned by a Deployment
- **THEN** that Deployment is shown as a link, and following it opens the Deployment's panel

### Requirement: Kind-specific sections
The object panel SHALL show structured sections for Node, ConfigMap, Secret,
PersistentVolumeClaim, ServiceAccount, ReplicaSet, Deployment, StatefulSet, DaemonSet, Job,
CronJob, Service, Ingress, Endpoints, EndpointSlice, NetworkPolicy, Namespace, PersistentVolume,
StorageClass, Role, ClusterRole, RoleBinding, ClusterRoleBinding and Event objects.

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
- **WHEN** a Node's panel is open in the structured view
- **THEN** a region below its sections lists, live and across every namespace, every pod whose
  `spec.nodeName` matches that node, as the Pods panel's own table scoped to the node, sharing
  the context's pod watch rather than starting another

#### Scenario: A node's pod is opened from the keyboard or the mouse
- **WHEN** a pod row in a Node's pods region is selected
- **THEN** Enter (`object_detail.node_pods.open_detail`) or `d` opens that pod's detail panel,
  `l` its logs, and `/` searches the rows, as in the Pods panel

#### Scenario: A node with no pods scheduled
- **WHEN** a Node's panel is open for a node with no pods scheduled to it
- **THEN** the region says no pods are scheduled on the node, rather than being omitted

#### Scenario: The pod list can't be retrieved
- **WHEN** the user may read the Node but the pod watch can't connect or is refused
- **THEN** the Node still loads, and the region shows the Pods panel's failed state rather than
  erroring the whole panel

### Requirement: A generic Status section for other kinds
Any kind without dedicated sections whose object carries a non-empty `status.conditions` array
SHALL show a generic Status section; any other kind with neither SHALL show metadata only.

#### Scenario: A Kustomization with no dedicated section gets a generic Status section
- **WHEN** a Kustomization's panel is open and its CRD has no kind-specific section defined
- **THEN** it shows a Status section with its derived message and its full conditions list, rather
  than metadata only

#### Scenario: A CRD with no conditions keeps metadata only
- **WHEN** a CRD object's panel is open and it has neither a kind-specific section nor
  `status.conditions`
- **THEN** it shows metadata only, unchanged from today

### Requirement: Secret values are hidden until revealed
The object panel SHALL NOT display a Secret value in its YAML view. In its structured view it SHALL
show each key's name and the value's size, with a reveal control per key that shows that one value
until it is hidden, the panel closes, or Hide Secret Values runs. A value SHALL NOT be written to
disk, logged, or placed in a window title, tab name, palette entry or notification.

#### Scenario: Structured view of a Secret
- **WHEN** a Secret's panel shows its structured view
- **THEN** it shows the Secret's type and each key with its size and a reveal control, and no value

#### Scenario: Revealing one value
- **WHEN** the user activates the reveal control on one key
- **THEN** that key's value is shown, and no other

#### Scenario: YAML view of a Secret
- **WHEN** a Secret's panel shows its YAML view
- **THEN** every `data` and `stringData` value, and any last-applied-configuration annotation, is
  replaced with a placeholder giving its size, even while a value is revealed in the structured view

#### Scenario: A revealed value stays out of saved state
- **WHEN** a value is revealed and the window's layout is saved
- **THEN** the saved layout contains no Secret value

### Requirement: The panel shows its object's events and YAML
The object panel SHALL show the events naming its object, newest first, and SHALL let the user
switch between the structured view and the object's YAML by keystroke or by clicking.

#### Scenario: Toggling to YAML by keystroke
- **WHEN** an object's panel has focus and the user presses the view-toggle key
- **THEN** the panel shows the object's YAML, and pressing it again returns to the structured view

#### Scenario: Events that can't be listed
- **WHEN** the user may read the object but not list events
- **THEN** the object still loads, and the events section says why it's empty

### Requirement: The panel survives its object's absence
If the object a panel is scoped to doesn't exist, the panel SHALL say so rather than erroring or
closing.

#### Scenario: Following a link to an object that doesn't exist
- **WHEN** the user follows a reference to an object that was deleted or never created
- **THEN** the panel opens and shows that the object doesn't exist

### Requirement: The YAML view scrolls and folds
In the pod detail and object detail panels, the YAML view SHALL scroll vertically when it is taller
than the panel and SHALL let the user fold and unfold each nested mapping or sequence, by keyboard
and by mouse.

#### Scenario: A large manifest
- **WHEN** an object's YAML is longer than the panel and the user folds `metadata.managedFields`
- **THEN** the view scrolls to every line, and the folded block shows as one line until unfolded

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

### Requirement: Large label and annotation values are shortened in the object viewer
The object panel's structured view SHALL shorten label and annotation chips exactly as the pod
detail panel does: a value longer than 100 characters or spanning more than one line shows its first
20 characters of the first line and an ellipsis, with the full value in a hover tooltip. A value the
panel redacts, such as a Secret's last-applied-configuration annotation, SHALL show only its
redacted placeholder in both the chip and the tooltip.

#### Scenario: An object's long annotation is shortened
- **WHEN** a Deployment's panel shows an annotation whose value is multi-line
- **THEN** its chip shows the key, the first 20 characters of the first line, and an ellipsis, and
  hovering it shows the full value

#### Scenario: A redacted annotation stays redacted
- **WHEN** a Secret's panel shows its last-applied-configuration annotation
- **THEN** neither the chip nor its tooltip contains any Secret value

### Requirement: Object detail follows its object live
The object detail panel SHALL reflect its object's current state while it is open, updating its
sections, metadata and YAML within a few seconds of a change in the cluster, without the user
reopening or refreshing it, and keeping the user's view state.

#### Scenario: A Deployment rolls out
- **WHEN** a Deployment's detail panel is open and the Deployment is scaled from 2 to 3 replicas
- **THEN** the panel's desired, ready and available replica counts update as the rollout proceeds

#### Scenario: Deleted while open
- **WHEN** an object's detail panel is open and the object is deleted
- **THEN** within a few seconds the panel says it was deleted and when, keeps its last known state
  visible as stale, and stays open

### Requirement: A derived status message appears near the top of the panel

The object panel SHALL show, near the top, a single derived status message for any object whose
`status.conditions` include a non-empty message, in addition to any conditions it already shows. The
message SHALL be the `Ready` condition's message when present and non-empty, toned Good for `True`,
Bad for `False` and Warning otherwise; else the most recently transitioned condition's non-empty
message, untoned. An object with no such message SHALL show no message field.

#### Scenario: An object without a message shows no message field
- **WHEN** an object's panel is open and none of its conditions has a non-empty message
- **THEN** the panel shows no message field

### Requirement: A Ready field beside the message

When an object has a `Ready` condition, the object panel SHALL show a Ready field with that
condition's status (`True`, `False` or `Unknown`), toned and marked like the list's Ready cell,
unless the kind's own section already shows readiness.

#### Scenario: A HelmRelease shows its Ready status
- **WHEN** a HelmRelease's panel is open and its `Ready` condition has `status: "False"`
- **THEN** the panel shows a Ready field reading False in its Bad tone, beside the message

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
