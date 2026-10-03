# Spec Delta

## ADDED Requirements

### Requirement: Init container states read as they are
The pod detail panel SHALL show each init container's actual state - waiting (with its reason and
message), running, or terminated (with exit code and reason) - and SHALL NOT show a waiting init
container as completed.

#### Scenario: A waiting init container
- **WHEN** a Pod's init container is waiting with reason `PodInitializing`
- **THEN** the panel shows that init container as Waiting with that reason, not as completed

### Requirement: Pod status is color-coded
The pod detail panel SHALL color the pod's phase and each container's state by severity using the
theme's semantic colors - success for Running and Succeeded, info for Pending and Waiting, danger for
Failed, Error and CrashLoopBackOff - and SHALL keep the state's text so it reads without color.

#### Scenario: A crashing container
- **WHEN** a container is waiting with reason `CrashLoopBackOff`
- **THEN** its state shows in the danger color with the text `CrashLoopBackOff`

### Requirement: Managed fields are collapsed by default
The Managed Fields tab SHALL show one disclosure row per field manager (manager, operation, time),
collapsed by default, expanding to that manager's fields, operable by keyboard as well as mouse.

#### Scenario: Expanding one manager
- **WHEN** the user opens the Managed Fields tab and expands the `kubectl-client-side-apply` row
- **THEN** only that manager's fields are shown, and the other managers stay collapsed
