## Purpose

Streaming a Pod container's logs into a panel, with follow mode and container selection for
multi-container Pods.

## ADDED Requirements

### Requirement: Stream a pod's logs

The application SHALL open a log panel for a selected Pod that streams that Pod's container log output
and appends new lines as they arrive.

#### Scenario: Open logs for a pod

- **WHEN** the user opens logs for a running Pod
- **THEN** the panel shows the recent log history and continues appending new lines as the container emits them

#### Scenario: Pod terminates while streaming

- **WHEN** the Pod being streamed is deleted or its container exits
- **THEN** the panel indicates the stream has ended and retains the lines already shown

#### Scenario: Log request fails

- **WHEN** the log request is rejected, for example the container has not started
- **THEN** the panel shows the failure reason rather than an empty view

### Requirement: Follow mode

The log panel SHALL support a follow mode that keeps the newest line in view, and SHALL suspend
following when the user scrolls up and resume it when the user returns to the bottom.

#### Scenario: Following the tail

- **WHEN** follow mode is on and new lines arrive
- **THEN** the view stays pinned to the newest line

#### Scenario: Scrolling up pauses follow

- **WHEN** the user scrolls up while following
- **THEN** the view stops auto-scrolling until the user scrolls back to the bottom or re-enables follow

### Requirement: Container selection

For a Pod with more than one container, the log panel SHALL let the user choose which container's logs
to stream and SHALL default to the first container.

#### Scenario: Switch container

- **WHEN** the user selects a different container in a multi-container Pod's log panel
- **THEN** the panel switches to streaming that container's logs

#### Scenario: Single-container pod

- **WHEN** the Pod has exactly one container
- **THEN** that container's logs stream without requiring the user to pick one
