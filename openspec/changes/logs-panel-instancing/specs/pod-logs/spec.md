# Spec Delta

## ADDED Requirements

### Requirement: Opening a pod's logs can open a new panel instead of reusing one
The application SHALL let the user open more than one pod's logs simultaneously, each in its own
panel, rather than being limited to a single Logs panel that always shows whichever pod was most
recently selected.

#### Scenario: Reuse is the default
- **WHEN** a user opens a second pod's logs with the default preference unchanged
- **THEN** the existing Logs panel retargets to the new pod's logs, matching current behavior

#### Scenario: Forcing a new panel
- **WHEN** a user opens a pod's logs with the new-panel override
- **THEN** a new panel opens for that pod's logs, and any already-open logs panel for a
  different pod is unaffected

#### Scenario: Reopening the same pod's forced-new logs focuses the existing one
- **WHEN** a user forces a new logs panel for a pod that already has one open
- **THEN** the existing panel for that pod is focused, not duplicated

### Requirement: The default behavior is configurable
The application SHALL let the user configure whether opening a pod's logs defaults to reusing a
single panel or always opening a new per-pod instance, and SHALL let a single invocation override
that default in the opposite direction.

#### Scenario: Default set to per-pod instancing
- **WHEN** the user's preference is set to always open a new instance, and they open a pod's logs
  without the override
- **THEN** a new per-pod panel opens rather than retargeting an existing one
