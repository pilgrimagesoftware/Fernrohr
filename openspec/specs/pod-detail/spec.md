## Purpose

A dockable panel showing one Pod's description or YAML, independent of the Pods list panel it
was opened from, so detail can be viewed, moved, and closed like any other panel.

## Requirements

### Requirement: Detail opens as its own panel

Requesting a Pod's description or YAML SHALL open a dockable panel scoped to that specific pod,
distinct from the Pods list panel it was requested from.

#### Scenario: Describing a pod opens a detail panel

- **WHEN** the user requests a pod's description from a Pods list
- **THEN** a new panel opens showing that pod's description, and the Pods list panel remains
  showing only its table

#### Scenario: Requesting YAML opens a detail panel

- **WHEN** the user requests a pod's YAML from a Pods list
- **THEN** a new panel opens showing that pod's YAML

### Requirement: Reselecting the same pod focuses its existing panel

Requesting detail for a pod that already has an open detail panel for the same cluster SHALL
focus that panel rather than opening a duplicate.

#### Scenario: Same pod requested twice

- **WHEN** a pod's detail panel is already open and the user requests that same pod's detail
  again
- **THEN** the existing panel is focused and no second panel opens

#### Scenario: Different pods each get their own panel

- **WHEN** the user requests detail for two different pods in the same cluster
- **THEN** two distinct detail panels are open, each scoped to its own pod

### Requirement: Detail panel survives its source pod's disappearance

If the pod a detail panel is scoped to no longer exists in the cluster, the panel SHALL show
that the pod no longer exists rather than erroring or closing unexpectedly.

#### Scenario: Pod deleted while its detail panel is open

- **WHEN** a pod is deleted from the cluster while its detail panel is open
- **THEN** the panel shows that the pod no longer exists, and remains open until the user closes
  it

### Requirement: Structured field view by default

The detail panel SHALL render a structured, labeled list of the pod's fields by default,
including at minimum: creation time, name, namespace, labels, annotations, controlling owner,
status, node, host and pod IPs, service account, QoS class, termination grace period,
tolerations, and conditions.

#### Scenario: Structured fields shown by default

- **WHEN** a pod's detail panel opens
- **THEN** it shows the structured field list, not raw YAML

#### Scenario: Labels and annotations render as discrete chips

- **WHEN** a pod's detail panel shows a pod with more than one label
- **THEN** each label renders as its own visually distinct chip, not a single run-on line of text

#### Scenario: Conditions render as colored status badges

- **WHEN** a pod's detail panel shows its conditions
- **THEN** each condition renders as a badge colored by whether its status is `True`

### Requirement: Raw YAML remains available as an alternate view

The detail panel SHALL let the user switch to viewing the pod's raw YAML manifest without
leaving the panel.

#### Scenario: Switching to YAML view

- **WHEN** the user toggles the detail panel's YAML view
- **THEN** the panel shows the pod's YAML manifest in place of the structured field list

### Requirement: Pod detail links the objects the pod references
The pod detail panel SHALL show, as `resource-links` references, the pod's namespace, each of its
owners, its node, its service account, every ConfigMap, Secret and PersistentVolumeClaim its
volumes name, its image pull secrets, and every ConfigMap and Secret its containers read through
`envFrom` or `valueFrom`.

#### Scenario: Each owner is its own link
- **WHEN** a pod has more than one owner reference
- **THEN** each owner is shown as its own reference, not one combined line of text

#### Scenario: Volume sources are references
- **WHEN** a pod mounts a volume backed by a ConfigMap, a Secret or a PersistentVolumeClaim
- **THEN** that ConfigMap, Secret or claim is shown as a reference in the volume's entry

#### Scenario: Container environment sources are references
- **WHEN** a container reads environment variables from a ConfigMap or Secret, whole
  (`envFrom`) or by key (`valueFrom`)
- **THEN** that container's entry shows each such ConfigMap or Secret as a reference, once
  per object even when several keys come from it

#### Scenario: Image pull secrets are references
- **WHEN** a pod names image pull secrets
- **THEN** each is shown as a reference in the pod's fields

#### Scenario: Following the node link from a pod
- **WHEN** the application can display Nodes and the user follows a pod's node link
- **THEN** that node's panel opens (or is focused) in the pod's cluster context

### Requirement: Structured fields are organized into tabs
The pod detail panel's structured field view SHALL organize its fields into tabs rather than one
continuous scroll, and SHALL let the user switch tabs by keystroke as well as by clicking.

#### Scenario: Switching tabs shows only that tab's fields
- **WHEN** the user switches to a tab
- **THEN** only that tab's fields are visible, and the panel does not require scrolling past
  unrelated fields to reach them

#### Scenario: Every field still appears somewhere
- **WHEN** every tab's fields are considered together
- **THEN** every field `pod_fields` projects for the pod appears in exactly one tab

### Requirement: The Events tab lists the pod's own events
The pod detail panel SHALL show, in an Events tab, the events whose involved object is this pod,
newest first, and SHALL report a failure to list them rather than showing an empty list.

#### Scenario: Events are this pod's, newest first
- **WHEN** the pod has events, including ones recorded only through `events.k8s.io/v1` fields
- **THEN** the Events tab lists them newest first with their age and repeat count, and lists no
  event belonging to another object of the same name

#### Scenario: Events cannot be listed
- **WHEN** the pod can be read but listing its events fails (for example, RBAC forbids it)
- **THEN** the pod's fields still load, and the Events tab says the events could not be listed
