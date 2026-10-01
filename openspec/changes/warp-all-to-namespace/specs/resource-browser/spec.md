## MODIFIED Requirements

### Requirement: Namespace scoping

A resource-browser panel SHALL let the user view a single namespace or all namespaces, SHALL persist the selection with the panel, and SHALL inherit the active context's default namespace when opened. A "warp all" action SHALL update every open namespaced panel in that context to the selected namespace and set that namespace as the context default.

#### Scenario: Warp all updates open panels

- **WHEN** the user invokes "Warp all to namespace" for a selected Pod in `team-a`
- **THEN** every open namespaced resource panel in the same context shows only `team-a`
- **AND** cluster-scoped panels are unchanged

#### Scenario: Restrict to one namespace

- **WHEN** the user selects namespace `kube-system`
- **THEN** the table shows only Pods in `kube-system`

#### Scenario: All namespaces

- **WHEN** the user selects "all namespaces"
- **THEN** the table shows Pods from every namespace the user can list

#### Scenario: Warp all affects future panels

- **WHEN** the context default is `team-a`
- **AND** the user opens another namespaced resource panel in that context
- **THEN** the new panel starts scoped to `team-a`

#### Scenario: Existing focused warp remains local

- **WHEN** the user invokes the existing focused-panel warp for `team-a`
- **THEN** only the focused panel changes namespace
- **AND** the context default and other panels are unchanged
