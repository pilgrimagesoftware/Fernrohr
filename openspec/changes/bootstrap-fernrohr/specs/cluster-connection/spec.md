## Purpose

Discovering Kubernetes contexts from the user's kubeconfig and establishing a working API connection
to one selected context, including API resource discovery for that connection.

## ADDED Requirements

### Requirement: Read-only kubeconfig context discovery

The application SHALL load Kubernetes contexts from the kubeconfig referenced by `$KUBECONFIG`, or
`~/.kube/config` when that variable is unset, and SHALL never modify the kubeconfig file.

#### Scenario: Contexts listed from default kubeconfig

- **WHEN** the application starts and `~/.kube/config` contains three contexts
- **THEN** all three contexts are listed by name for the user to choose from

#### Scenario: KUBECONFIG override honored

- **WHEN** `$KUBECONFIG` points to a specific file
- **THEN** contexts are read from that file instead of `~/.kube/config`

#### Scenario: Missing kubeconfig is handled

- **WHEN** no kubeconfig file exists at the resolved path
- **THEN** the application reports that no contexts were found and remains usable

#### Scenario: Kubeconfig is never written

- **WHEN** the user connects to, disconnects from, or switches contexts
- **THEN** the kubeconfig file's contents and modification time are unchanged

### Requirement: Connect a single context

The application SHALL connect to one user-selected context by building a Kubernetes API client from
that context's cluster, user, and credentials, and SHALL surface connection success or a descriptive
failure.

#### Scenario: Successful connection

- **WHEN** the user selects a reachable context with valid credentials
- **THEN** the application establishes an API client and marks the context as connected

#### Scenario: Unreachable API server

- **WHEN** the selected context's API server cannot be reached
- **THEN** the application shows a connection error naming the server and does not mark the context connected

#### Scenario: Credential plugin failure

- **WHEN** the context relies on an exec credential plugin that exits non-zero
- **THEN** the application reports the credential failure with the plugin's error output

### Requirement: API resource discovery

On connecting, the application SHALL run Kubernetes API discovery for that connection and make the set
of available API groups, versions, and resource kinds queryable within the application.

#### Scenario: Discovery populates available kinds

- **WHEN** a connection is established to a cluster
- **THEN** the discovered resource kinds for that connection include the core kinds such as Pod, and
  discovery completes without blocking the user interface
