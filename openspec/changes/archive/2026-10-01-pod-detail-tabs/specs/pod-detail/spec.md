# Spec Delta

## ADDED Requirements

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
