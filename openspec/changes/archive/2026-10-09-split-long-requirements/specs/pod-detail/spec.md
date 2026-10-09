# Spec Delta

## MODIFIED Requirements

### Requirement: Large ConfigMap values start collapsed

In the Configuration tab, a ConfigMap value longer than 100 characters or spanning more than one
line SHALL be shown collapsed by default, as the first 20 characters of its first line followed by
an ellipsis.

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

### Requirement: Pod detail follows its pod's deletion live

While its detail panel is open, a pod's deletion SHALL be reflected live. Once the pod has a
deletion timestamp, the panel SHALL show it as Terminating with its remaining grace period.

#### Scenario: Deleted while open
- **WHEN** a pod's detail panel is open and the pod is deleted
- **THEN** the panel first shows Terminating, then says the pod was deleted and when, keeps its last
  known fields visible as stale, and still lists its events

#### Scenario: Recreated under the same name
- **WHEN** a StatefulSet pod's detail panel is open and the pod is deleted and recreated as `web-0`
- **THEN** the panel switches to the new `web-0`, shows that it replaced the deleted pod, and its
  fields follow the new pod live

## ADDED Requirements

### Requirement: A collapsed ConfigMap value has an expand control

Each collapsed ConfigMap value SHALL have a control placed directly beside the key's name,
reachable by Tab and operable with Enter, Space or a click, that expands the value to its full
contents and collapses it again.

#### Scenario: Expand and collapse from the keyboard

- **WHEN** a large value is collapsed and the user tabs to its control and presses Enter
- **THEN** the value is shown in full, and pressing Enter again collapses it

### Requirement: ConfigMap values collapse again when the tab is shown again

Large ConfigMap values SHALL start collapsed again each time the Configuration tab is shown
again.

#### Scenario: Showing the tab again

- **WHEN** the user expands a large value, shows another tab, then shows the Configuration tab again
- **THEN** the value is collapsed again

### Requirement: Secret values are never collapsed

Secret values in the Configuration tab SHALL NOT be collapsed: they SHALL stay hidden until
revealed, and a revealed value SHALL be shown in full.

#### Scenario: A revealed Secret value is shown in full

- **WHEN** a Secret key holds a long value and the user reveals it
- **THEN** the value is shown in full, with no collapse control

### Requirement: Pod detail keeps a deleted pod's last state

Once its pod is gone, a pod's detail panel SHALL say within a few seconds that the pod was
deleted and when, SHALL keep showing its last known state marked as stale rather than clearing it,
SHALL keep its Events tab listing the pod's events, and SHALL stay open until the user closes it.

#### Scenario: Deleted while open

- **WHEN** a pod's detail panel is open and the pod is deleted
- **THEN** the panel says the pod was deleted and when, keeps its last known fields visible as stale, still lists its events, and stays open

### Requirement: Pod detail switches to a pod recreated under the same name

If a new pod appears under a deleted pod's name with a different uid, the deleted pod's detail
panel SHALL switch to the new pod and show that it replaced the deleted one.

#### Scenario: Recreated under the same name

- **WHEN** a StatefulSet pod's detail panel is open and the pod is deleted and recreated as `web-0`
- **THEN** the panel switches to the new `web-0` and shows that it replaced the deleted pod
