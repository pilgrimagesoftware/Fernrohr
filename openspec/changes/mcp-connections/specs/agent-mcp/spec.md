# Spec Delta

## MODIFIED Requirements

### Requirement: Connect context tool
The MCP server SHALL provide a tool that connects a kubeconfig context for the requesting MCP session without adding it to any window. A context no window or session holds SHALL be connected and held by that session, and the Connections window SHALL open or come forward showing it. A context already connected SHALL be held by the session too, without connecting again and without asking approval.

#### Scenario: Connect a context that is not open
- **WHEN** an agent asks to connect `staging`, nothing holds it, and the user approves
- **THEN** `staging` connects, held by the agent's session only, no window gains it, and the Connections window shows it with the agent as its holder

#### Scenario: Context already connected
- **WHEN** an agent asks to connect a context that a window or another session already holds
- **THEN** no second connection is made and no approval is asked, the agent's session also holds it, and the result reports its current state

#### Scenario: Context already in the window
- **WHEN** an agent asks to connect a context the frontmost window already holds
- **THEN** that window is focused as before, and the agent's session also holds the context

#### Scenario: Unknown context
- **WHEN** an agent asks to connect a name that is not in the kubeconfig
- **THEN** the MCP response is an unknown-context error and nothing is asked or connected

### Requirement: Connecting requires approval
Before connecting a context that no window or MCP session holds, Fernrohr SHALL ask the user to approve, naming the context and its bound tunnel if any. A denied, timed-out, or abandoned request SHALL connect nothing.

#### Scenario: User denies a connection
- **WHEN** an agent asks to connect `prod` and the user denies it
- **THEN** no connection, tunnel, or credential flow starts, and the result is denied

## ADDED Requirements

### Requirement: MCP session holds end with the session
Each MCP client SHALL have one session with Fernrohr for as long as its server process runs, identified by the client's reported name. When the session ends, because the client exits, the server process stops, or Fernrohr stops serving MCP, every connection hold it took SHALL be released.

#### Scenario: Client exits
- **WHEN** an agent that connected `staging` exits and no window holds `staging`
- **THEN** `staging` disconnects and leaves the Connections window

#### Scenario: Stop serving with the last window
- **WHEN** the Agent access setting stops the MCP server with the last window, and the last window closes while an agent holds `staging`
- **THEN** the agent's session ends and `staging` disconnects

### Requirement: Agent connections survive the last window closing
When the last window closes and Fernrohr keeps running with the MCP server serving, connections held by MCP sessions SHALL stay open, and read tools SHALL keep answering for them.

#### Scenario: Read after closing every window
- **WHEN** an agent holds `staging`, the user closes every window, and Fernrohr keeps serving MCP
- **THEN** the agent's next list of `staging` resources succeeds without reconnecting or asking approval
