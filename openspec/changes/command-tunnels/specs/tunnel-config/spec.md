# Spec Delta

## MODIFIED Requirements

### Requirement: Define and manage named tunnels

The application SHALL provide a Tunnels panel where the user can create, edit, rename, and delete
named tunnel configurations. Each tunnel SHALL be one of two kinds, chosen in the editor: an SSH
tunnel or a command tunnel. An SSH tunnel describes only how to reach a bastion: a host (a
hostname or an SSH client configuration alias), a port, a login user, an authentication method
(the SSH client's own configuration and agent, or a private key held in the OS keychain), an
optional ordered jump-host list, and keepalive settings. A command tunnel is described by the
settings in "Command tunnel settings". An SSH tunnel SHALL NOT name a forward target; the target
is taken from the connecting context. The Tunnels panel SHALL NOT assign contexts to tunnels.

#### Scenario: Create a tunnel

- **WHEN** the user creates a tunnel named `qa-bastion` with a host, user, and key-based auth in the Tunnels panel
- **THEN** the tunnel appears in the tunnel list and is offered as a choice in every context's tunnel selector

#### Scenario: Create a command tunnel

- **WHEN** the user creates a tunnel named `qa-iap`, chooses the command kind, and enters a command line
- **THEN** the tunnel appears in the tunnel list marked as a command tunnel and is offered as a choice in every context's tunnel selector

#### Scenario: Rename and edit

- **WHEN** the user edits `qa-bastion`'s host and renames it
- **THEN** existing context bindings continue to reference the same tunnel under its new name

#### Scenario: Change a tunnel's kind

- **WHEN** the user switches an existing SSH tunnel to the command kind and saves it
- **THEN** the tunnel keeps its identity and context bindings, and the next connection through it runs the command

#### Scenario: Delete a bound tunnel

- **WHEN** the user deletes a tunnel that one or more contexts are bound to
- **THEN** the user is warned which contexts will fall back to a direct connection, and on confirmation those bindings are removed

#### Scenario: Invalid tunnel is rejected

- **WHEN** the user saves an SSH tunnel with an empty host or user, or a port outside 1-65535
- **THEN** the tunnel is not saved and the offending field is marked with the reason

### Requirement: Test a tunnel

The Tunnels panel SHALL let the user test a tunnel without connecting any context. Testing an SSH
tunnel SHALL establish an SSH session to the bastion with the tunnel's settings, without forwarding
any port, and report success or the SSH client's failure output. Testing a command tunnel SHALL run
its command until the tunnel is ready or its startup timeout expires, then stop the command, and
report success or the command's recent output.

#### Scenario: Test succeeds

- **WHEN** the user tests a tunnel whose bastion accepts the configured user and authentication
- **THEN** the panel reports the tunnel as reachable

#### Scenario: Test fails

- **WHEN** the user tests a tunnel whose bastion cannot be reached or rejects authentication
- **THEN** the panel reports the failure with the SSH client's error output, and no context connection is affected

#### Scenario: Command tunnel test succeeds

- **WHEN** the user tests a command tunnel whose command starts listening on its local port within the startup timeout
- **THEN** the panel reports the tunnel as reachable, and the command is no longer running afterward

#### Scenario: Command tunnel test fails

- **WHEN** the user tests a command tunnel whose command exits, or does not listen before the startup timeout
- **THEN** the panel reports the failure with the command's recent output, and no process from the test is left running

## ADDED Requirements

### Requirement: Command tunnel settings

A command tunnel SHALL carry:
- a command line, which may contain the placeholder `{port}`
- a mode, either `proxy` (the default: the local port offers an HTTP proxy) or `forward` (the local
  port reaches the API server directly)
- an optional fixed local port
- a startup timeout, 30 seconds by default

The command line SHALL be split into arguments using POSIX shell quoting rules. A backslash
followed by a line break SHALL be treated as a line continuation. The command SHALL NOT be passed to
a shell, so pipes, redirections and variable expansion are not interpreted. Every `{port}` SHALL be
replaced with the tunnel's local port: the fixed local port when one is set, or otherwise a free
port allocated at start.

Saving SHALL be refused, with the offending field marked, when:
- the command is empty or has unbalanced quotes
- the command has no `{port}` and no fixed local port is set
- the fixed local port is outside 1-65535
- the startup timeout is not a positive number of seconds

#### Scenario: Placeholder receives the allocated port

- **WHEN** a command tunnel with command `ssh -N -L{port}:127.0.0.1:8888 bastion` and no fixed port starts and is allocated port 53124
- **THEN** the process runs with argument `-L53124:127.0.0.1:8888`

#### Scenario: Fixed port without a placeholder

- **WHEN** a command tunnel's command has `-L8888:127.0.0.1:8888` hard-coded and its fixed local port is 8888
- **THEN** the tunnel saves, runs the command unchanged, and uses local port 8888

#### Scenario: Pasted multi-line command

- **WHEN** the user pastes a command split over several lines with trailing backslashes, and with quoted arguments
- **THEN** it is parsed as one command, with each quoted argument kept as a single argument without its quotes

#### Scenario: No way to know the port

- **WHEN** the user saves a command tunnel whose command has no `{port}` and that has no fixed local port
- **THEN** the tunnel is not saved and the command field is marked with the reason

### Requirement: Tunnels without a kind load as SSH

The application SHALL load a tunnel from `tunnels.toml` that has no kind as an SSH tunnel, with its
fields and context bindings unchanged. A command tunnel's command line SHALL be persisted as an
ordinary non-secret field.

#### Scenario: Existing file loads unchanged

- **WHEN** `tunnels.toml` was written by a build that had no command tunnels
- **THEN** every tunnel loads as an SSH tunnel and its context bindings work as before

#### Scenario: Command tunnel round-trips

- **WHEN** the user defines a command tunnel and relaunches the application
- **THEN** the tunnel is present with the same command line, mode, fixed local port, and startup timeout
