# Spec Delta

## MODIFIED Requirements

### Requirement: Define and manage named tunnels

The application SHALL provide a Tunnels panel where the user can create, edit, rename, and delete
named tunnel configurations. Each tunnel SHALL be one of three kinds, chosen in the editor: an SSH
tunnel, a command tunnel, or a manual tunnel. A command tunnel is described by the settings in
"Command tunnel settings". A manual tunnel is described by the settings in "Manual tunnel settings".

#### Scenario: Create a tunnel

- **WHEN** the user creates a tunnel named `qa-bastion` with a host, user, and key-based auth in the Tunnels panel
- **THEN** the tunnel appears in the tunnel list and is offered as a choice in every context's tunnel selector

#### Scenario: Create a command tunnel

- **WHEN** the user creates a tunnel named `qa-iap`, chooses the command kind, and enters a command line
- **THEN** the tunnel appears in the tunnel list marked as a command tunnel and is offered as a choice in every context's tunnel selector

#### Scenario: Create a manual tunnel

- **WHEN** the user creates a tunnel named `corp-vpn` and chooses the manual kind
- **THEN** the tunnel appears in the tunnel list marked as a manual tunnel and is offered as a choice in every context's tunnel selector

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

### Requirement: Command tunnel settings

A command tunnel SHALL carry:
- a command line, which may contain the placeholder `{port}`
- a mode, either `proxy` (the default: the local port offers an HTTP proxy) or `forward` (the local
  port reaches the API server directly)
- an optional fixed local port
- a startup timeout, 30 seconds by default

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

## ADDED Requirements

### Requirement: SSH tunnel settings

An SSH tunnel SHALL describe only how to reach a bastion: a host (a hostname or an SSH client
configuration alias), a port, a login user, an authentication method (the SSH client's own
configuration and agent, or a private key held in the OS keychain), an optional ordered jump-host
list, and keepalive settings. An SSH tunnel SHALL NOT name a forward target; the target is taken
from the connecting context.

#### Scenario: No forward target in an SSH tunnel

- **WHEN** the user edits an SSH tunnel
- **THEN** the editor asks for the bastion's host, port, user, authentication, jump hosts, and
  keepalive, and for no forward target

### Requirement: The Tunnels panel does not bind contexts

The Tunnels panel SHALL NOT assign contexts to tunnels.

#### Scenario: No binding in the Tunnels panel

- **WHEN** the user creates or edits a tunnel in the Tunnels panel
- **THEN** the panel offers no way to assign a context to the tunnel

### Requirement: A command tunnel's command line is split without a shell

A command tunnel's command line SHALL be split into arguments using POSIX shell quoting rules. A
backslash followed by a line break SHALL be treated as a line continuation. The command SHALL NOT be
passed to a shell, so pipes, redirections and variable expansion are not interpreted.

#### Scenario: A continued line with quoted arguments

- **WHEN** a command tunnel's command is split over several lines with trailing backslashes and has
  quoted arguments
- **THEN** it is parsed as one command, each quoted argument kept as a single argument without its
  quotes

### Requirement: A command tunnel's port placeholder is filled in

Every `{port}` in a command tunnel's command line SHALL be replaced with the tunnel's local port:
the fixed local port when one is set, or otherwise a free port allocated at start.

#### Scenario: A fixed port fills the placeholder

- **WHEN** a command tunnel whose command contains `{port}` has a fixed local port of 8888 and
  starts
- **THEN** the command runs with 8888 in place of every `{port}`

### Requirement: Invalid command tunnel settings are refused

Saving a command tunnel SHALL be refused, with the offending field marked, when:
- the command is empty or has unbalanced quotes
- the command has no `{port}` and no fixed local port is set
- the fixed local port is outside 1-65535
- the startup timeout is not a positive number of seconds

#### Scenario: Unbalanced quotes

- **WHEN** the user saves a command tunnel whose command has an unclosed quote
- **THEN** the tunnel is not saved and the command field is marked with the reason
