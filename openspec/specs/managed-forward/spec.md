## Purpose

A long-lived local network forward that Fernrohr starts, supervises, health-checks, and tears down on
demand, presenting a uniform state and lifecycle whether it is backed by an SSH tunnel or a
Kubernetes port-forward.

## Requirements

### Requirement: Uniform forward lifecycle and state

A managed forward SHALL expose an observable state that is one of Disconnected, Connecting, Up, or
Reconnecting, and SHALL allocate a free local loopback port when it starts unless a specific local
port is requested.

#### Scenario: Start allocates a port and reaches Up

- **WHEN** a managed forward is started with no explicit local port
- **THEN** it binds a free `127.0.0.1` port, transitions Connecting then Up, and reports the bound port

#### Scenario: Start failure is terminal until retried

- **WHEN** a managed forward cannot establish its underlying transport
- **THEN** it transitions to Disconnected with a descriptive reason and does not hold the local port

#### Scenario: Explicit local port in use

- **WHEN** a managed forward is started with an explicit local port that is already bound
- **THEN** it fails to start with a reason naming the port conflict

### Requirement: Health checking and reconnection

A managed forward SHALL periodically verify the forward is still carrying traffic, and on failure
SHALL transition to Reconnecting and attempt to re-establish with backoff, returning to Up on success
without changing its allocated local port.

#### Scenario: Transient drop recovers

- **WHEN** an Up forward's underlying transport drops briefly
- **THEN** it transitions to Reconnecting, re-establishes, returns to Up, and keeps the same local port

#### Scenario: Sustained failure keeps retrying

- **WHEN** an Up forward's transport stays unavailable
- **THEN** it remains in Reconnecting with increasing backoff and continues attempting until stopped

### Requirement: Reference-counted sharing

A managed forward SHALL be reference counted: it starts on the first acquire, is shared by all
holders, and stops when the last holder releases it.

#### Scenario: Shared by two holders

- **WHEN** two holders acquire the same managed forward
- **THEN** exactly one underlying forward exists and both observe the same state and port

#### Scenario: Stops on last release

- **WHEN** the last holder releases a managed forward
- **THEN** the underlying transport is closed and the local port is freed

### Requirement: SSH tunnel implementation

An SSH-backed managed forward SHALL forward its local port to a remote host and port through an SSH
bastion, optionally via a chain of jump hosts, using an SSH tunnel configuration.

#### Scenario: Forward through a bastion

- **WHEN** an SSH tunnel to bastion `b` forwarding to `api.internal:6443` reaches Up
- **THEN** a TCP connection to the tunnel's local port reaches `api.internal:6443` as seen from `b`

#### Scenario: Jump-host chain

- **WHEN** the SSH tunnel configuration lists an ordered set of jump hosts before the bastion
- **THEN** the tunnel is established through that chain in order

#### Scenario: Authentication failure surfaces

- **WHEN** the SSH bastion rejects authentication
- **THEN** the forward reports an authentication failure reason and does not reach Up

### Requirement: Kubernetes port-forward implementation

A Kubernetes-backed managed forward SHALL forward its local port to a Pod or Service port on a
connected cluster using the Kubernetes port-forward API.

#### Scenario: Forward to a service port

- **WHEN** a Kubernetes port-forward to `svc/example:80` on a connected cluster reaches Up
- **THEN** a request to the tunnel's local port is served by that Service

#### Scenario: Target disappears

- **WHEN** the forwarded Pod is deleted while the forward is Up
- **THEN** the forward transitions to Reconnecting and recovers if a matching target returns, or keeps retrying otherwise

### Requirement: Command tunnel implementation

A command-backed managed forward SHALL run a command tunnel's command as a child process, giving it
the user's login-shell `PATH` so that command-line tools installed for the user are found even when
the application was not started from a terminal. The child's standard input SHALL stay open for the
child's whole lifetime, so a command that opens an interactive session keeps its forward running.

#### Scenario: Ready when the port listens

- **WHEN** a command forward's process starts and begins accepting connections on its local port within the startup timeout
- **THEN** the forward transitions Connecting then Up and reports that port

#### Scenario: Command exits before ready

- **WHEN** a command forward's process exits before its local port accepts a connection
- **THEN** the forward transitions to Disconnected with a reason that includes the exit status and the command's recent output

#### Scenario: Startup timeout

- **WHEN** a command forward's process is still running but its local port does not accept a connection within the startup timeout
- **THEN** the process group is terminated and the forward transitions to Disconnected with a timeout reason and the command's recent output

#### Scenario: Process dies while Up

- **WHEN** an Up command forward's process exits
- **THEN** the forward transitions to Reconnecting and re-runs the command with backoff, keeping the same local port

#### Scenario: Stop leaves nothing behind

- **WHEN** the last holder releases a command forward whose command started child processes of its own
- **THEN** the command and all of its descendants are terminated and the local port is freed

#### Scenario: Tool found outside a terminal launch

- **WHEN** the application was launched from the desktop rather than a terminal, and the command names a tool that is only on the user's login-shell `PATH`
- **THEN** the command starts successfully

### Requirement: A command forward is Up only when its port accepts connections

A command-backed managed forward SHALL reach Up only when the process is running and a TCP
connection to its local loopback port succeeds within the startup timeout. While the forward is Up,
the process exiting or the port refusing connections SHALL count as a transport failure for health
checking and reconnection.

#### Scenario: A refused port while Up

- **WHEN** an Up command forward's local port starts refusing connections
- **THEN** the forward treats it as a transport failure and reconnects

### Requirement: Stopping a command forward leaves no processes behind

Stopping a command-backed managed forward SHALL terminate the command's whole process group: first
gracefully, then forcibly if it has not exited after a short grace period, so that no descendant
processes are left running.

#### Scenario: A command that ignores the graceful stop

- **WHEN** a command forward is stopped and its command has not exited after the grace period
- **THEN** its whole process group is terminated forcibly and no descendant process is left running

### Requirement: A command forward's failures carry its recent output

A command-backed managed forward SHALL keep the most recent lines of the command's output and
include them in any failure reason.

#### Scenario: Output in the failure reason

- **WHEN** a command forward fails
- **THEN** its failure reason includes the most recent lines of the command's output
