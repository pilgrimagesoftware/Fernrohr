# Spec Delta

## ADDED Requirements

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
In the Configuration tab, a ConfigMap value longer than 20 characters or spanning more than one line
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
- **WHEN** a ConfigMap key holds a value of 20 characters or fewer on one line
- **THEN** the full value is shown with no expand control

#### Scenario: Expanding one value
- **WHEN** the user tabs to one value's expand control and presses Space
- **THEN** that value is shown in full and every other large value stays collapsed

#### Scenario: Secret values are hidden, not collapsed
- **WHEN** a Secret key holds a long value
- **THEN** the value is hidden with no collapse control, and revealing it shows it in full
