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
