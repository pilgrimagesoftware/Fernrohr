# Spec Delta

## MODIFIED Requirements

### Requirement: Each pod's logs open in a panel of their own

The application SHALL by default open a pod's logs in a Logs panel for that pod alone, so the user
can keep several pods' logs open side by side, rather than retargeting a single Logs panel to
whichever pod was opened last. Opening the same pod's logs again SHALL focus its panel rather than
add a second, and opening another container of that pod SHALL switch its panel to that container.

#### Scenario: Per-pod panels are the default

- **WHEN** a user opens one pod's logs and then another pod's, with the default preference
- **THEN** each pod's logs are in a panel of their own, and the first panel is unaffected

#### Scenario: Reopening a pod's logs focuses its panel

- **WHEN** a user opens the logs of a pod whose logs panel is already open
- **THEN** that panel is focused, not duplicated

#### Scenario: Another container of an open pod

- **WHEN** a user opens a different container's logs of a pod whose logs panel is open
- **THEN** that pod's panel switches to the container, and no second panel opens

#### Scenario: A pod's panel ignores later selections

- **WHEN** a user selects another pod without opening its logs
- **THEN** each pod's logs panel keeps showing its own pod

## ADDED Requirements

### Requirement: A pod's logs panel stays on its pod

A pod's Logs panel SHALL stay on its pod when other pods are selected later, and SHALL restore as
that pod's panel from a saved layout.

#### Scenario: Restored from a saved layout

- **WHEN** a layout holding a pod's Logs panel is saved and later loaded
- **THEN** the restored panel shows that pod's logs

#### Scenario: Later selections leave it alone

- **WHEN** the user selects another pod without opening its logs
- **THEN** the open pod's Logs panel keeps showing its own pod
