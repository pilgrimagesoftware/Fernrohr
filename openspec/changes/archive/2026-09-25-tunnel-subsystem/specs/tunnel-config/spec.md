## Purpose

Managing named SSH tunnel configurations - their host, credentials, and options - with non-secret
fields persisted to disk and secrets held only in the operating system keychain.

## ADDED Requirements

### Requirement: Define and manage named tunnels

The application SHALL let the user create, edit, and delete named SSH tunnel configurations, each with
a bastion host and port, a login user, an authentication method, an optional ordered jump-host list,
and keepalive settings.

#### Scenario: Create a tunnel

- **WHEN** the user creates a tunnel named `qa-bastion` with a host, user, and key-based auth
- **THEN** the tunnel appears in the tunnel list and can be bound to a context

#### Scenario: Rename and edit

- **WHEN** the user edits `qa-bastion`'s host and renames it
- **THEN** existing context bindings continue to reference the same tunnel under its new name

#### Scenario: Delete a bound tunnel

- **WHEN** the user deletes a tunnel that one or more contexts are bound to
- **THEN** the user is warned which contexts will be unbound, and on confirmation those bindings are removed

### Requirement: Persist non-secret fields only

The application SHALL persist tunnel configurations to `tunnels.toml` in the state directory
containing no secret material, and SHALL reload them on the next launch.

#### Scenario: Round-trip across restart

- **WHEN** the user defines two tunnels and relaunches the application
- **THEN** both tunnels are present with their non-secret fields intact

#### Scenario: No secrets on disk

- **WHEN** a tunnel uses password or key-passphrase authentication
- **THEN** `tunnels.toml` contains no password, passphrase, or private-key bytes

### Requirement: Secrets in the OS keychain

The application SHALL store tunnel secrets (passwords, private-key passphrases) in the operating
system keychain, keyed by tunnel identity, and retrieve them when establishing the tunnel.

#### Scenario: Secret saved and reused

- **WHEN** the user supplies a passphrase for a tunnel and later starts that tunnel in a new session
- **THEN** the passphrase is read from the keychain without prompting the user again

#### Scenario: No keychain daemon available

- **WHEN** the platform has no available keychain or Secret Service daemon
- **THEN** the application prompts for the secret for the current session instead of failing

#### Scenario: Secret removed with the tunnel

- **WHEN** the user deletes a tunnel
- **THEN** its keychain entry is removed
