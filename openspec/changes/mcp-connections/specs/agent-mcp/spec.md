# Spec Delta

## MODIFIED Requirements

### Requirement: Connect context tool
The MCP server SHALL provide a tool that connects a kubeconfig context for the requesting MCP session without adding it to any window. Once connected the session SHALL hold the context, and the Connections window SHALL open or come forward showing it. A context a user window already holds SHALL be held by the session without connecting again and without asking approval.

#### Scenario: Connect a context that is not open
- **WHEN** an agent asks to connect `staging`, nothing holds it, and the user approves
- **THEN** `staging` connects, held by the agent's session only, no window gains it, and the Connections window shows it with the agent as its holder

#### Scenario: Context a window holds
- **WHEN** an agent asks to connect a context that a user window already holds
- **THEN** no second connection is made and no approval is asked, the agent's session also holds it, and the result reports its current state

#### Scenario: Context already in the window
- **WHEN** an agent asks to connect a context the frontmost window already holds
- **THEN** that window is focused as before, and the agent's session also holds the context

#### Scenario: Context only another agent holds
- **WHEN** an agent asks to connect `staging`, which only a different MCP session holds
- **THEN** the user is asked to approve as for a new connection, and only on approval does this session hold `staging`, reusing the existing connection

#### Scenario: Unknown context
- **WHEN** an agent asks to connect a name that is not in the kubeconfig
- **THEN** the MCP response is an unknown-context error and nothing is asked or connected

### Requirement: Connecting requires approval
Before an MCP session holds a context that no user window holds, Fernrohr SHALL ask the user to approve, naming the context and its bound tunnel if any. Approval SHALL be bound to the session that asked: another session's approval of the same context SHALL never count. A denied, timed-out, or abandoned request SHALL connect nothing and grant no hold.

#### Scenario: User denies a connection
- **WHEN** an agent asks to connect `prod` and the user denies it
- **THEN** no connection, tunnel, or credential flow starts, and the result is denied

#### Scenario: Denied while another agent holds it
- **WHEN** an agent asks to connect `prod`, which another agent's session holds, and the user denies it
- **THEN** the asking session holds nothing, and the other agent's hold is unchanged

## ADDED Requirements

### Requirement: Every request belongs to a live MCP session
Each MCP client SHALL have one session with Fernrohr for as long as its server process runs. Fernrohr SHALL reject a tool request that names no session, or a session that is unknown or has ended, with an error telling the user to restart their MCP client, and SHALL perform nothing for it.

#### Scenario: Older adapter
- **WHEN** a request arrives from an MCP server process that opened no session
- **THEN** the response is an error saying to restart the MCP client, and nothing is read, asked or connected

#### Scenario: Ended session
- **WHEN** a request names a session that has ended, because Fernrohr restarted or stopped serving MCP
- **THEN** Fernrohr refuses it and nothing is read, asked or connected; the MCP server process opens a new session and retries once, and the new session holds nothing, so a connect needs fresh approval and a read of a context only the old session held reports it not open

#### Scenario: Reopening fails
- **WHEN** the MCP server process can't open a new session after its old one ended
- **THEN** the response is the restart error, and nothing is read, asked or connected

### Requirement: Session identifiers are secret
A session identifier SHALL be 128 bits from the operating system's random source, SHALL never be logged, shown in the UI, or returned to an MCP client in tool results, and SHALL never be reused.

#### Scenario: Identifier stays out of logs
- **WHEN** a session opens, makes requests, and ends with debug logging enabled
- **THEN** no log line contains its identifier

### Requirement: MCP session holds end with the session
When a session ends, because the client exits, the server process stops, or Fernrohr stops serving MCP, each of its connection holds SHALL become ownerless. An ownerless hold SHALL keep its connection open for a short grace period, SHALL NOT be claimable by any session, and SHALL be released when the period ends. A restarted client SHALL get a new session that needs approval again.

#### Scenario: Client exits
- **WHEN** an agent that connected `staging` exits and no window holds `staging`
- **THEN** after the grace period `staging` disconnects and leaves the Connections window

#### Scenario: Client restarts within the grace period
- **WHEN** an agent that held `staging` restarts and asks to connect `staging` before the grace period ends
- **THEN** the user is asked to approve, and on approval the new session holds `staging` reusing the still-open connection

#### Scenario: Stop serving with the last window
- **WHEN** the Agent access setting stops the MCP server with the last window, and the last window closes while an agent holds `staging`
- **THEN** the agent's session ends and `staging` disconnects after the grace period

### Requirement: Reads need the session's own access
Read tools SHALL answer for a context only when a user window holds it or the requesting session holds it. For a context held only by other sessions, a read SHALL report the context as not connected, so the agent has to connect it, and be approved, first. Reads of window-held contexts SHALL be unchanged.

#### Scenario: Another agent's connection
- **WHEN** agent B lists resources in `staging`, which only agent A's session holds
- **THEN** the result reports `staging` as not connected, and nothing is read

#### Scenario: Window-held context
- **WHEN** an agent lists resources in a context a user window holds, without having connected it
- **THEN** the result lists the resources as before

### Requirement: Agent connections survive the last window closing
When the last window closes and Fernrohr keeps running with the MCP server serving, connections held by MCP sessions SHALL stay open, and read tools SHALL keep answering for them.

#### Scenario: Read after closing every window
- **WHEN** an agent holds `staging`, the user closes every window, and Fernrohr keeps serving MCP
- **THEN** the agent's next list of `staging` resources succeeds without reconnecting or asking approval
