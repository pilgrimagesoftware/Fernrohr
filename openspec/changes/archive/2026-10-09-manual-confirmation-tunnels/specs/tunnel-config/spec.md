# Spec Delta

## MODIFIED Requirements

### Requirement: Define and manage named tunnels

The application SHALL provide a Tunnels panel where the user can create, edit, rename, and delete
named tunnel configurations. Each tunnel SHALL be one of three kinds, chosen in the editor: an SSH
tunnel, a command tunnel, or a manual tunnel. An SSH tunnel describes only how to reach a bastion: a host (a
hostname or an SSH client configuration alias), a port, a login user, an authentication method
(the SSH client's own configuration and agent, or a private key held in the OS keychain), an
optional ordered jump-host list, and keepalive settings. A command tunnel is described by the
settings in "Command tunnel settings". A manual tunnel is described by the settings in "Manual
tunnel settings". An SSH tunnel SHALL NOT name a forward target; the target
is taken from the connecting context. The Tunnels panel SHALL NOT assign contexts to tunnels.

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
