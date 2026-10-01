## MODIFIED Requirements

### Requirement: Define and manage named tunnels

The application SHALL provide a Tunnels panel where the user can create, edit, rename, and delete
named SSH tunnel configurations. Each tunnel describes only how to reach a bastion: a host (a
hostname or an SSH client configuration alias), a port, a login user, an authentication method
(the SSH client's own configuration and agent, or a private key held in the OS keychain), an
optional ordered jump-host list, and keepalive settings. A tunnel SHALL NOT name a forward target;
the target is taken from the connecting context. The Tunnels panel SHALL NOT assign contexts to
tunnels.

#### Scenario: Create a tunnel

- **WHEN** the user creates a tunnel named `qa-bastion` with a host, user, and key-based auth in the Tunnels panel
- **THEN** the tunnel appears in the tunnel list and is offered as a choice in every context's tunnel selector

#### Scenario: Rename and edit

- **WHEN** the user edits `qa-bastion`'s host and renames it
- **THEN** existing context bindings continue to reference the same tunnel under its new name

#### Scenario: Delete a bound tunnel

- **WHEN** the user deletes a tunnel that one or more contexts are bound to
- **THEN** the user is warned which contexts will fall back to a direct connection, and on confirmation those bindings are removed

#### Scenario: Invalid tunnel is rejected

- **WHEN** the user saves a tunnel with an empty host or user, or a port outside 1-65535
- **THEN** the tunnel is not saved and the offending field is marked with the reason

## ADDED Requirements

### Requirement: Tunnel usage is shown read-only

The Tunnels panel SHALL show, for each tunnel, how many contexts are bound to it and whether it is
currently running, without offering any control there to change which contexts use it.

#### Scenario: Usage count

- **WHEN** contexts `cluster-a` and `cluster-b` are bound to `qa-bastion`
- **THEN** the Tunnels panel shows `qa-bastion` in use by two contexts

#### Scenario: Running state

- **WHEN** a connected context is using `qa-bastion`
- **THEN** the Tunnels panel shows `qa-bastion` as running, and as idle once no connection uses it

### Requirement: Test a tunnel

The Tunnels panel SHALL let the user test a tunnel, which SHALL establish an SSH session to the
bastion with the tunnel's settings, without forwarding any port or connecting any context, and
report success or the SSH client's failure output.

#### Scenario: Test succeeds

- **WHEN** the user tests a tunnel whose bastion accepts the configured user and authentication
- **THEN** the panel reports the tunnel as reachable

#### Scenario: Test fails

- **WHEN** the user tests a tunnel whose bastion cannot be reached or rejects authentication
- **THEN** the panel reports the failure with the SSH client's error output, and no context connection is affected

### Requirement: Legacy tunnel entries load

The application SHALL load a `tunnels.toml` whose tunnels still carry a forward target from an
earlier version, ignoring those fields, and SHALL write the file without them the next time it
saves.

#### Scenario: Old file loads

- **WHEN** `tunnels.toml` contains a tunnel with `remote_host` and `remote_port` fields
- **THEN** the tunnel and its context bindings load normally, and the next save omits both fields
