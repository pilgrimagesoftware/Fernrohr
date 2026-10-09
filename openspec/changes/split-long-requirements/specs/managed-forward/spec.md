# Spec Delta

## MODIFIED Requirements

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

## ADDED Requirements

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
