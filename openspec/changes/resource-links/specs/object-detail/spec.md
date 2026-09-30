# Spec Delta

## Purpose

A dockable panel showing one Kubernetes object of any kind the cluster reports, opened by
following a reference to it: its metadata, the sections that matter for its kind, the events
naming it, and its YAML, without ever showing a Secret's values.

## ADDED Requirements

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

### Requirement: Secret values are never shown
The object panel SHALL NOT display any Secret value, in its structured view or its YAML view.
It SHALL show each key's name and the value's size in bytes instead.

#### Scenario: Structured view of a Secret
- **WHEN** a Secret's panel shows its structured view
- **THEN** it shows the Secret's type and each key with its size, and no value

#### Scenario: YAML view of a Secret
- **WHEN** a Secret's panel shows its YAML view
- **THEN** every `data` and `stringData` value, and any last-applied-configuration annotation, is
  replaced with a placeholder giving its size

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
