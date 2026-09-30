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
