# Spec Delta

## Purpose

Gives each Kubernetes resource kind a recognizable full-colour icon, shown wherever the kind
appears, so panels, tabs and links can be told apart at a glance.

## ADDED Requirements

### Requirement: Each kind has a full-colour icon
The application SHALL map every built-in Kubernetes kind covered by the Kubernetes community icon
set to that set's full-colour icon, bundled with the application. A kind the set doesn't cover
SHALL get a fallback icon in the same style: one for containers, one for custom-resource
instances, and one generic kind icon for anything else. No kind SHALL be drawn without an icon.

#### Scenario: A Pod's icon
- **WHEN** a Pod is shown anywhere an icon appears
- **THEN** it shows the Kubernetes set's Pod icon in full colour

#### Scenario: A custom resource
- **WHEN** a CRD instance is shown
- **THEN** it shows the custom-resource fallback icon

#### Scenario: A container
- **WHEN** a container is listed in pod detail
- **THEN** it shows the container fallback icon

### Requirement: Icons appear where kinds do
An icon SHALL be shown beside the kind's name in panel title bars and tabs, the Resource panel's
kind list, Configuration-tab and object-viewer cards, and resource links, sized to the adjacent
text and scaled with it.

#### Scenario: Panel tab
- **WHEN** a window has a Pods panel and a ConfigMap panel open as tabs
- **THEN** each tab shows its kind's icon before its title

#### Scenario: Icon scales with text
- **WHEN** the user increases the text size
- **THEN** icons grow with the text beside them

### Requirement: Icons are decorative to keyboard and accessibility
Icons SHALL NOT be focusable or change any keyboard route, and SHALL carry the kind's name as their
accessible label only where the adjacent text doesn't already name the kind.

#### Scenario: Tab order unchanged
- **WHEN** the user tabs through a panel with icons
- **THEN** focus visits the same controls in the same order as without icons

### Requirement: The icon set is credited
The About window SHALL credit the Kubernetes community icon set, and the set's licence SHALL ship
with the application.

#### Scenario: Credit in About
- **WHEN** the user opens the About window
- **THEN** its credits name the Kubernetes icon set and its licence
