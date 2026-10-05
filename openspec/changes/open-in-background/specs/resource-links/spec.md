# Spec Delta

## ADDED Requirements

### Requirement: Following a link in the background

Following a link with the platform modifier held (`cmd`-click on macOS, `ctrl`-click elsewhere),
or with a middle-click, SHALL open the referenced object's panel in the same cluster context as
an inactive tab, leaving keyboard focus in the panel the link was followed from. If the object's
panel is already open there, nothing SHALL change.

#### Scenario: Background link

- **WHEN** the user `cmd`-clicks a pod detail panel's link to its owning DaemonSet
- **THEN** the DaemonSet's panel opens as an inactive tab, and the pod detail panel keeps focus

#### Scenario: Background link to an open object

- **WHEN** the user middle-clicks a link to an object whose panel is already open
- **THEN** no panel opens, no tab is activated, and focus does not move
