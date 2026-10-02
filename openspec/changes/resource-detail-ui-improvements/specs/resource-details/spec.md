# Spec Delta

## MODIFIED Requirements

### Requirement: Resource detail pane displays relevant fields for the selected resource
The resource detail pane SHALL present resource fields in a structured, readable format. The detail pane SHALL support copy actions for key values and improved YAML rendering for inspectability.

#### Scenario: Viewing Pod detail shows container and init container states clearly
- **WHEN** a Pod with init containers is selected and an init container is waiting
- **THEN** the detail pane SHALL display the init container status in a clear, non-misleading form indicating it is waiting (not completed)

#### Scenario: YAML view supports scrolling and folding
- **WHEN** viewing a large manifest in the YAML tab of the detail pane
- **THEN** the YAML view SHALL be vertically scrollable and SHALL support folding/collapsing of nested YAML structures

#### Scenario: Resource name can be copied via action
- **WHEN** a resource is selected and the user invokes the "Copy Resource Name" action
- **THEN** the resource's name SHALL be copied to the clipboard

#### Scenario: Per-field values expose copy affordance
- **WHEN** hovering over specific fields in the detail pane (e.g., Pod container image, ConfigMap key, ConfigMap value)
- **THEN** a copy button SHALL appear on hover and clicking it SHALL copy that field's value to the clipboard

#### Scenario: Status values are color-coded for quick scanning
- **WHEN** displaying status values in the detail pane (including Pod status)
- **THEN** status values SHALL be visually distinguished by color to reflect their state (e.g., running/succeeded vs pending/waiting/failed/terminated)

#### Scenario: Managed fields use collapsible disclosure
- **WHEN** viewing managed fields in the detail pane
- **THEN** managed fields SHALL be rendered using a collapsible disclosure control (expand/collapse) similar to ConfigMap key/value presentation to reduce visual noise
