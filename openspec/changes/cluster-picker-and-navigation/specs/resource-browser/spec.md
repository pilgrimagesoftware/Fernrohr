# Spec Delta

## ADDED Requirements

### Requirement: Resource kind navigation

A connected window SHALL provide a way to switch which resource kind its active panel
displays, choosing from the kinds the application currently supports, without closing
and reopening the window's connection.

#### Scenario: Switch from Pods to Logs

- **WHEN** a window is showing a Pods panel for a connected cluster
- **AND** the user navigates to Logs
- **THEN** the panel is replaced by a Logs view for the same connection, without
  reconnecting or losing the cluster session

#### Scenario: Navigation reflects the connected cluster's discovery

- **WHEN** a window is connected to a cluster
- **THEN** the navigation only offers resource kinds the application supports and that
  cluster's API discovery confirms are available
