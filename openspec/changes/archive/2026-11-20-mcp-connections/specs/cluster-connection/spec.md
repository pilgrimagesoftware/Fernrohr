# Spec Delta

## MODIFIED Requirements

### Requirement: Connections shared across windows

The application SHALL keep at most one connection per context, shared by every window and every
MCP session that uses that context. A connection, and any tunnel it holds, SHALL stay open while at
least one window or MCP session holds the context, and SHALL be shut down when the last holder
disconnects the context, closes, or ends.

#### Scenario: Shared by two windows

- **WHEN** two windows both use `cluster-a`
- **THEN** one connection, one set of watches, and at most one tunnel forward serve both

#### Scenario: One window lets go

- **WHEN** one of those windows disconnects `cluster-a`
- **THEN** the other window's `cluster-a` panels keep receiving live updates without reconnecting

#### Scenario: Last window lets go

- **WHEN** the last window using `cluster-a` disconnects it or closes, and no MCP session holds it
- **THEN** `cluster-a`'s connection and watches stop, and its tunnel forward is released

#### Scenario: Window closes while an agent holds the context

- **WHEN** the only window using `cluster-a` closes while an MCP session also holds `cluster-a`
- **THEN** `cluster-a` stays connected, and the agent's next read of it succeeds without reconnecting

#### Scenario: Retry keeps an agent's hold

- **WHEN** `cluster-a` failed to connect while only an agent holds it, and the user retries it
- **THEN** the same connection reconnects, the agent's hold is kept, and the agent's next read of `cluster-a` succeeds

#### Scenario: Agent's session ends last

- **WHEN** an MCP session that is the last holder of `cluster-a` ends
- **THEN** `cluster-a`'s connection and watches stop, and its tunnel forward is released
