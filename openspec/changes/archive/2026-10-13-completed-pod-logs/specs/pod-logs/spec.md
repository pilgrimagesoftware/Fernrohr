# Spec Delta

## MODIFIED Requirements

### Requirement: Stream a pod's logs

The application SHALL open a log panel for a selected Pod that streams that Pod's container log output
and appends new lines as they arrive. For a container that has already terminated, the application
SHALL read its complete log without following it and SHALL show that output.

#### Scenario: Open logs for a pod

- **WHEN** the user opens logs for a running Pod
- **THEN** the panel shows the recent log history and continues appending new lines as the container emits them

#### Scenario: Pod terminates while streaming

- **WHEN** the Pod being streamed is deleted or its container exits
- **THEN** the panel retains the lines already shown and indicates after them that the stream has ended

#### Scenario: Log request fails

- **WHEN** the log request is rejected, for example the container has not started
- **THEN** the panel shows the failure reason rather than an empty view

## ADDED Requirements

### Requirement: Completed pods' logs stay readable

The log panel SHALL show the full log output of a container that has already terminated, such as
the pod of a completed Job, and SHALL mark the end of the stream after that output rather than in
place of it.

#### Scenario: Open logs for a completed pod

- **WHEN** the user opens logs for a pod whose container has terminated
- **THEN** the panel shows the container's complete output, followed by a marker that the log stream ended

#### Scenario: Completed container wrote nothing

- **WHEN** the user opens logs for a terminated container that wrote no output
- **THEN** the panel says the container wrote no log output, rather than showing a failure

#### Scenario: Pod no longer exists

- **WHEN** the user opens logs for a pod that has been deleted
- **THEN** the panel shows that the pod was not found
