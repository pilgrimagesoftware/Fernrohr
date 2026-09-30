# Spec Delta

## Purpose

How the application uses colour and surface to give its views structure: one accent for what is
interactive and current, layered surfaces, status colour, and a contrast floor.

## ADDED Requirements

### Requirement: One accent for interactive and current elements
Links, the selected row, the focused tab, focus rings and primary buttons SHALL share one accent
colour, distinct from body text: the system accent colour where the platform provides one, the
theme's blue otherwise.

#### Scenario: A selected row
- **WHEN** a table row is selected
- **THEN** it is tinted with the accent colour, not only with a neutral fill

#### Scenario: A link at rest
- **WHEN** a followable reference is shown and the pointer is elsewhere
- **THEN** it is drawn in the accent colour

### Requirement: Layered surfaces without framing borders
Panel chrome (a panel's heading and hint bar, table headers, section headers) and the cards inside
detail views SHALL each sit on a fill distinct from the window background. The application SHALL
NOT draw a border framing a panel's content.

#### Scenario: A panel's heading
- **WHEN** a detail panel is shown
- **THEN** its heading and hint bar sit on a fill distinct from its body

### Requirement: Status is shown in colour
The Pods table SHALL colour each pod's status by its health: running in the good tone, pending in
the warning tone, failed or crash-looping in the bad tone, completed in the neutral tone.

#### Scenario: A crash-looping pod
- **WHEN** a pod's container is waiting with reason `CrashLoopBackOff`
- **THEN** that pod's status is shown in the bad tone

### Requirement: Contrast floor
In light and dark mode, text SHALL meet a 4.5:1 contrast ratio against every surface it is drawn
on, and the accent and status colours SHALL meet 3:1 against their backgrounds.

#### Scenario: A light system accent in light mode
- **WHEN** the system accent colour fails 3:1 against the light background
- **THEN** the application uses a darkened accent that meets it
