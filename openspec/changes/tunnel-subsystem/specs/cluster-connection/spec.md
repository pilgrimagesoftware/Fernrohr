## MODIFIED Requirements

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

## ADDED Requirements

### Requirement: Recoverable connection interruptions

The application SHALL treat a bound tunnel dropping, or an exec credential-plugin `401` requiring
re-authentication, as a recoverable pause of that connection rather than a disconnect: its watch
streams SHALL be suspended and then resumed and reconverged once the tunnel returns to Up or
re-authentication succeeds.

#### Scenario: Tunnel drop pauses then resumes watches

- **WHEN** a connected context's bound tunnel transitions from Up to Reconnecting while watch streams are active
- **THEN** those watch streams are suspended, the connection shows a paused state, and when the tunnel returns to Up the streams resume and the resource views converge to current cluster state

#### Scenario: Credential re-authentication does not drop watches

- **WHEN** the API server returns `401` and the context's exec credential plugin can mint a fresh token
- **THEN** the client re-authenticates, the active watch streams are resumed rather than torn down, and no panel is closed

#### Scenario: Unrecoverable failure becomes a disconnect

- **WHEN** a bound tunnel enters a sustained failure that does not recover and the user chooses to disconnect
- **THEN** the connection moves to disconnected and its watches are released
