# Spec Delta

## ADDED Requirements

### Requirement: Previous container logs

For a Pod whose current container has restarted, the log panel SHALL provide a "previous logs" mode
that streams the log output of that container's last-terminated instance instead of its current one.

#### Scenario: Toggle to previous logs

- **WHEN** the user toggles previous logs for a container that has restarted at least once
- **THEN** the panel shows the log output of that container's last-terminated instance

#### Scenario: Toggle back to current logs

- **WHEN** the user toggles previous logs off
- **THEN** the panel resumes streaming the container's current log output, including any lines
  emitted while previous logs was shown

#### Scenario: No previous instance exists

- **WHEN** the user toggles previous logs for a container that has never restarted
- **THEN** the panel indicates there is no previous instance rather than showing an empty view
