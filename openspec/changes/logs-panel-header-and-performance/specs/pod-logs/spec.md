# Spec Delta

## ADDED Requirements

### Requirement: Log panel identifies its pod and container
A log panel SHALL show which pod and which container it is streaming in its title.

#### Scenario: Title shows pod and container
- **WHEN** a log panel is streaming a container's logs
- **THEN** its title includes that pod's name and that container's name

### Requirement: Log rendering does not degrade with line count
A log panel's scrolling performance SHALL NOT degrade as the number of received log lines grows
unboundedly - only lines within or near the visible range are rendered at a time.

#### Scenario: Scrolling a long-running log stream
- **WHEN** a log panel has received several thousand lines
- **THEN** scrolling remains responsive, without a per-scroll cost proportional to total line
  count
