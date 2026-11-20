# Spec Delta

## MODIFIED Requirements

### Requirement: Disconnect a context from a window

Each status bar capsule SHALL offer a disconnect action. Before acting, the application SHALL ask
for confirmation, stating how many panels in this window will close and, if other windows or MCP
sessions still use the context, that it stays connected there, naming each agent. On confirmation
it SHALL close every panel in this window that uses the context and remove the capsule. If no
context remains, the window SHALL return to the cluster picker.

#### Scenario: Confirm and close panels

- **WHEN** the user disconnects `cluster-b`, which has three panels in this window, and confirms
- **THEN** those three panels close, the capsule is removed, and panels for other contexts are unchanged

#### Scenario: Cancel

- **WHEN** the user cancels the confirmation
- **THEN** no panel closes and the capsule remains

#### Scenario: Context used elsewhere

- **WHEN** the user disconnects `cluster-a` while another window also uses it
- **THEN** the confirmation says `cluster-a` stays connected in one other window, and after confirming the other window keeps its live `cluster-a` panels

#### Scenario: Context an agent still uses

- **WHEN** the user disconnects `cluster-a` from the only window using it while an agent also holds it
- **THEN** the confirmation says `cluster-a` stays connected for that agent, naming it, and after confirming the agent's reads of `cluster-a` keep working

#### Scenario: Last context

- **WHEN** the user disconnects the only context in a window and confirms
- **THEN** the window shows the cluster picker

## ADDED Requirements

### Requirement: Status bar capsule marks agent-held contexts
A status bar capsule SHALL show an agent icon when an MCP session also holds its context, with a
tooltip naming each such agent by its self-reported name, labelled as such. The icon SHALL not
replace or recolor the capsule's state icon.

#### Scenario: Agent shares a window's context
- **WHEN** a window uses `cluster-a` and an agent named `assistant` also holds it
- **THEN** the window's `cluster-a` capsule shows the agent icon, and its tooltip says an agent reporting itself as `assistant` also uses it

#### Scenario: Agent lets go
- **WHEN** that agent's session ends
- **THEN** the capsule's agent icon disappears and `cluster-a` stays connected in the window
