# Spec Delta

## Purpose

The screen that lets a user choose which Kubernetes context to connect to before any
resource panel opens, turning a bare window into a usable app on first run.

## ADDED Requirements

### Requirement: Picker shown when a window has no panels

The application SHALL show a cluster picker in place of the panel workspace whenever a
window has no open or restored panels, and SHALL replace the picker with the panel
workspace once a connection succeeds.

#### Scenario: First launch shows the picker

- **WHEN** the application starts with no saved workspace state
- **THEN** the main window shows the cluster picker instead of an empty panel area

#### Scenario: Window with restored panels skips the picker

- **WHEN** a window's saved workspace state includes at least one panel
- **THEN** that window opens directly into its restored panel workspace, not the picker

### Requirement: Context list from kubeconfig

The picker SHALL list every context available from the kubeconfig source that
`cluster-connection` loads, showing at minimum each context's name and cluster.

#### Scenario: Contexts are listed

- **WHEN** the kubeconfig has three contexts
- **THEN** the picker lists all three by name

#### Scenario: No contexts available

- **WHEN** the kubeconfig has zero contexts, or none can be read
- **THEN** the picker shows a message explaining no contexts are available instead of an
  empty, unexplained list

### Requirement: Selecting a context connects

The picker SHALL let the user select one listed context and initiate a connection to it,
showing connection progress and, on success, opening the panel workspace for that
connection.

#### Scenario: Successful connection

- **WHEN** the user selects a context and it connects successfully
- **THEN** the picker is replaced by the panel workspace, landing on a default resource
  view for that connection

#### Scenario: Connection failure is shown on the picker

- **WHEN** the user selects a context and the connection attempt fails
- **THEN** the picker remains visible, shows the descriptive failure from
  `cluster-connection`, and lets the user pick a different context or retry
