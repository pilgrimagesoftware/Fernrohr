## ADDED Requirements

### Requirement: Connections shared across windows

The application SHALL keep at most one connection per context, shared by every window that uses
that context. A connection, and any tunnel it holds, SHALL stay open while at least one window uses
the context, and SHALL be shut down when the last such window disconnects the context or closes.

#### Scenario: Shared by two windows

- **WHEN** two windows both use `greedygoat`
- **THEN** one connection, one set of watches, and at most one tunnel forward serve both

#### Scenario: One window lets go

- **WHEN** one of those windows disconnects `greedygoat`
- **THEN** the other window's `greedygoat` panels keep receiving live updates without reconnecting

#### Scenario: Last window lets go

- **WHEN** the last window using `greedygoat` disconnects it or closes
- **THEN** `greedygoat`'s connection and watches stop, and its tunnel forward is released
