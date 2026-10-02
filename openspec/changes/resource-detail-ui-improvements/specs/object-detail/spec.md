# Spec Delta

## ADDED Requirements

### Requirement: The YAML view scrolls and folds
In the pod detail and object detail panels, the YAML view SHALL scroll vertically when it is taller
than the panel and SHALL let the user fold and unfold each nested mapping or sequence, by keyboard
and by mouse.

#### Scenario: A large manifest
- **WHEN** an object's YAML is longer than the panel and the user folds `metadata.managedFields`
- **THEN** the view scrolls to every line, and the folded block shows as one line until unfolded

### Requirement: Values can be copied
The pod detail and object detail panels SHALL offer a Copy Resource Name command (palette and key)
for the panel's object, and a copy control on copyable values - container images, ConfigMap keys and
values, and similar single values - that appears on hover and is reachable by keyboard. A Secret
value SHALL be copyable only while revealed.

#### Scenario: Copying the name
- **WHEN** a detail panel has focus and the user runs Copy Resource Name
- **THEN** the object's name is on the clipboard

#### Scenario: Copying an image
- **WHEN** the user hovers a container's image and clicks its copy control
- **THEN** the full image reference is on the clipboard
