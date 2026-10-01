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

### Requirement: Configuration tab
The pod detail panel SHALL have a Configuration tab listing every ConfigMap and Secret the pod
references, through volumes, projected volumes, `envFrom`, `valueFrom` and `imagePullSecrets`, once
per object. Each entry SHALL link to the object, say where the pod uses it, and show its contents:
a ConfigMap's keys with their values, and a Secret's keys with their sizes.

#### Scenario: A mounted ConfigMap's contents
- **WHEN** a pod mounts ConfigMap `app-config` and the user opens the Configuration tab
- **THEN** the tab shows `app-config` as a link, the volume and mount path it's used through, and
  each of its keys with its value

#### Scenario: One entry per object
- **WHEN** a pod mounts a Secret and also reads a key from it through `valueFrom`
- **THEN** the tab shows that Secret once, listing both uses

#### Scenario: A referenced object that doesn't exist
- **WHEN** a pod references a Secret marked `optional` that doesn't exist
- **THEN** that Secret's entry says it doesn't exist, and the other entries still show their
  contents

#### Scenario: Contents load when the tab is opened
- **WHEN** a pod detail panel opens
- **THEN** no ConfigMap or Secret is read until the user opens the Configuration tab

### Requirement: Secret values are hidden until revealed, one at a time
In the Configuration tab, each Secret key SHALL have a reveal control, reachable by Tab and operable
with Enter or Space, which shows that one value until it is hidden again. A revealed value SHALL be
hidden when the user leaves the tab or closes the panel, and by a Hide Secret Values command.

#### Scenario: Revealing one value
- **WHEN** the user activates the reveal control on key `password` of Secret `db`
- **THEN** that value is shown, and no other Secret value is

#### Scenario: Leaving the tab hides values
- **WHEN** a value is revealed and the user switches to another tab and back
- **THEN** the value is hidden again

#### Scenario: Hiding every revealed value
- **WHEN** two values are revealed and the user runs Hide Secret Values
- **THEN** both are hidden

#### Scenario: Revealing by keyboard
- **WHEN** the user tabs to a reveal control and presses Space
- **THEN** that value is revealed

### Requirement: Tab keys follow tab order
The pod detail panel's tab keys SHALL be positional: 1 Overview, 2 Containers, 3 Configuration, 4
Volumes, 5 Events, 6 Managed Fields.

#### Scenario: The Configuration tab key
- **WHEN** a pod detail panel has focus and the user presses `3`
- **THEN** the Configuration tab is shown

### Requirement: Large ConfigMap values start collapsed
In the Configuration tab, a ConfigMap value longer than 100 characters or spanning more than one line
SHALL be shown collapsed by default, as its first 20 characters of its first line followed by an
ellipsis, with a control placed directly beside the key's name, reachable by Tab and operable with
Enter, Space or a click, that expands it to its full contents and collapses it again. Values SHALL start collapsed again when the tab is
shown again. Secret values SHALL NOT be collapsed: they stay hidden until revealed, and a revealed
value is shown in full.

#### Scenario: A long value starts collapsed
- **WHEN** a ConfigMap key holds a multi-line value and the user opens the Configuration tab
- **THEN** that key shows the first 20 characters of the value's first line and an ellipsis, with an
  expand control beside the key's name, not at the far edge of the row

#### Scenario: A short value has no control
- **WHEN** a ConfigMap key holds a value of 100 characters or fewer on one line
- **THEN** the full value is shown with no expand control

#### Scenario: Expanding one value
- **WHEN** the user tabs to one value's expand control and presses Space
- **THEN** that value is shown in full and every other large value stays collapsed

#### Scenario: Secret values are hidden, not collapsed
- **WHEN** a Secret key holds a long value
- **THEN** the value is hidden with no collapse control, and revealing it shows it in full
