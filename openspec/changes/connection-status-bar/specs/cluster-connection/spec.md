## MODIFIED Requirements

### Requirement: Recoverable connection interruptions

The application SHALL treat a bound tunnel dropping, or an exec credential-plugin `401` requiring
re-authentication, as a recoverable pause of that connection rather than a disconnect: its watch
streams SHALL be suspended and then resumed and reconverged once the tunnel returns to Up or
re-authentication succeeds. While paused, the connection's state, reason, and elapsed time SHALL be
shown in the status bar of each window with panels for that context, and not inside individual
panels. Paused panels SHALL stay open and keep showing their last data.

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
