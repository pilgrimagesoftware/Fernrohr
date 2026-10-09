# Spec Delta

## MODIFIED Requirements

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

#### Scenario: Connection through a bound tunnel

- **WHEN** the user connects a context bound to a tunnel
- **THEN** the application waits for the tunnel to be Up, then builds the client against the tunnel's
  local port with the TLS server name pinned to the original API server host, and the certificate validates

#### Scenario: Bound tunnel fails to start

- **WHEN** the user connects a context whose bound tunnel cannot reach Up
- **THEN** the connection fails with the tunnel's failure reason and no client is built

### Requirement: Recoverable connection interruptions

The application SHALL treat a bound tunnel dropping, or an exec credential-plugin `401` requiring
re-authentication, as a recoverable pause of that connection rather than a disconnect: its watch
streams SHALL be suspended and then resumed and reconverged once the tunnel returns to Up or
re-authentication succeeds.

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

## ADDED Requirements

### Requirement: Connecting through a bound tunnel

When the selected context is bound to a tunnel, the application SHALL acquire that tunnel and wait
for it to be Up before building the client, SHALL build the client against the tunnel's local
loopback address with the TLS server name pinned to the context's original API server host, and
SHALL release the tunnel when the last cluster session using it disconnects.

#### Scenario: Client built through the tunnel

- **WHEN** the user connects a context bound to a tunnel
- **THEN** the client is built only once the tunnel is Up, against the tunnel's local loopback
  address, with the TLS server name pinned to the original API server host

#### Scenario: Tunnel released with its last session

- **WHEN** the last cluster session using a tunnel disconnects
- **THEN** the application releases that tunnel

### Requirement: A paused connection is shown in the status bar

While a connection is paused, its state, reason, and elapsed time SHALL be shown in the status bar
of each window with panels for that context, and not inside individual panels. Paused panels SHALL
stay open and keep showing their last data.

#### Scenario: Panels stay open while paused

- **WHEN** a context's connection pauses while panels in a window show it
- **THEN** the window's status bar shows the pause's state, reason, and elapsed time, and the panels
  stay open showing their last data
