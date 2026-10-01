## Purpose

Discovering Kubernetes contexts from the user's kubeconfig and establishing a working API connection
to one selected context, including API resource discovery for that connection.

## Requirements

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
failure. When the selected context is bound to a tunnel, the application SHALL acquire that tunnel and
wait for it to be Up before building the client, SHALL build the client against the tunnel's local
loopback address with the TLS server name pinned to the context's original API server host, and SHALL
release the tunnel when the last cluster session using it disconnects.

#### Scenario: Successful connection

- **WHEN** the user selects a reachable context with valid credentials
- **THEN** the application establishes an API client and marks the context as connected

#### Scenario: Unreachable API server

- **WHEN** the selected context's API server cannot be reached
- **THEN** the application shows a connection error naming the server and does not mark the context connected

#### Scenario: Credential plugin failure

- **WHEN** the context relies on an exec credential plugin that exits non-zero
- **THEN** the application reports the credential failure with the plugin's error output

#### Scenario: Connection through a bound tunnel

- **WHEN** the user connects a context bound to a tunnel
- **THEN** the application waits for the tunnel to be Up, then builds the client against the tunnel's
  local port with the TLS server name pinned to the original API server host, and the certificate validates

#### Scenario: Bound tunnel fails to start

- **WHEN** the user connects a context whose bound tunnel cannot reach Up
- **THEN** the connection fails with the tunnel's failure reason and no client is built

### Requirement: API resource discovery

On connecting, the application SHALL run Kubernetes API discovery for that connection and make the set
of available API groups, versions, and resource kinds queryable within the application. When the
connection is routed through a tunnel, discovery SHALL run over the tunneled client.

#### Scenario: Discovery populates available kinds

- **WHEN** a connection is established to a cluster
- **THEN** the discovered resource kinds for that connection include the core kinds such as Pod, and
  discovery completes without blocking the user interface

#### Scenario: Discovery over a tunneled connection

- **WHEN** a connection is established through a tunnel
- **THEN** discovery completes using the tunneled client and populates the available kinds

### Requirement: Recoverable connection interruptions

The application SHALL treat a bound tunnel dropping, or an exec credential-plugin `401` requiring
re-authentication, as a recoverable pause of that connection rather than a disconnect: its watch
streams SHALL be suspended and then resumed and reconverged once the tunnel returns to Up or
re-authentication succeeds. While paused, the connection's state, reason, and elapsed time SHALL be
shown in the status bar of each window with panels for that context, and not inside individual
panels. Paused panels SHALL stay open and keep showing their last data.

#### Scenario: Tunnel drop pauses then resumes watches

- **WHEN** a connected context's bound tunnel transitions from Up to Reconnecting while watch streams are active
- **THEN** those watch streams are suspended, the window status bar shows that context as reconnecting, and when the tunnel returns to Up the streams resume and the resource views converge to current cluster state

#### Scenario: Credential re-authentication does not drop watches

- **WHEN** the API server returns `401` and the context's exec credential plugin can mint a fresh token
- **THEN** the client re-authenticates, the active watch streams are resumed rather than torn down, and no panel is closed

#### Scenario: Unrecoverable failure becomes a disconnect

- **WHEN** a bound tunnel enters a sustained failure that does not recover and the user chooses to disconnect
- **THEN** the connection moves to disconnected and its watches are released

#### Scenario: One status for many panels

- **WHEN** three panels in one window show the same paused context
- **THEN** the pause is shown once, in that window's status bar, and none of the three panels shows its own pause message

### Requirement: Connections shared across windows

The application SHALL keep at most one connection per context, shared by every window that uses
that context. A connection, and any tunnel it holds, SHALL stay open while at least one window uses
the context, and SHALL be shut down when the last such window disconnects the context or closes.

#### Scenario: Shared by two windows

- **WHEN** two windows both use `cluster-a`
- **THEN** one connection, one set of watches, and at most one tunnel forward serve both

#### Scenario: One window lets go

- **WHEN** one of those windows disconnects `cluster-a`
- **THEN** the other window's `cluster-a` panels keep receiving live updates without reconnecting

#### Scenario: Last window lets go

- **WHEN** the last window using `cluster-a` disconnects it or closes
- **THEN** `cluster-a`'s connection and watches stop, and its tunnel forward is released
