# Spec Delta

## ADDED Requirements

### Requirement: Viewing logs from the pod detail panel

The pod detail panel SHALL have a command that opens the Logs panel for the pod it is scoped to,
reachable by keystroke and from the command palette while the panel has focus, without requiring
the user to return to the Pods list.

#### Scenario: Opening logs from an open detail panel

- **WHEN** the user invokes the view-logs command while a pod detail panel has focus
- **THEN** the Logs panel opens (or focuses, if already open) for that same pod

#### Scenario: Multi-container pod still offers container selection

- **WHEN** the user opens logs from the detail panel of a pod with more than one container
- **THEN** the Logs panel defaults to the first container and lets the user choose another, the
  same as opening logs from the Pods list does
