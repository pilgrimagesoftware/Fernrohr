# Spec Delta

## ADDED Requirements

### Requirement: Connect context tool
The MCP server SHALL provide a tool that connects a kubeconfig context in the frontmost Fernrohr window, adding it to that window as the status bar's add-context control does, or opening a window for it when none is open. A context the window already holds SHALL be focused without connecting again.

#### Scenario: Connect a context that is not open
- **WHEN** an agent asks to connect `staging` and the user approves
- **THEN** the frontmost window gains `staging`, opens its Pods panel, and makes it the selected cluster

#### Scenario: Context already in the window
- **WHEN** an agent asks to connect a context the frontmost window already holds
- **THEN** no second connection is made and no approval is asked, and the result reports its current state

#### Scenario: Unknown context
- **WHEN** an agent asks to connect a name that is not in the kubeconfig
- **THEN** the MCP response is an unknown-context error and nothing is asked or connected

### Requirement: Connecting requires approval
Before connecting a context the window does not already hold, Fernrohr SHALL ask the user to approve, naming the context and its bound tunnel if any. A denied, timed-out, or abandoned request SHALL connect nothing.

#### Scenario: User denies a connection
- **WHEN** an agent asks to connect `prod` and the user denies it
- **THEN** no connection, tunnel, or credential flow starts, and the result is denied

### Requirement: Connect reports the settled state
After an approved connection starts, the tool SHALL wait up to a bounded time for it to settle and SHALL report connected, waiting for its tunnel, awaiting a manual tunnel's confirmation, failed with a safe reason, or still connecting when the wait ends.

#### Scenario: Manual tunnel needs the user
- **WHEN** an agent connects a context bound to a manual tunnel that is not yet confirmed
- **THEN** the result reports that it is awaiting confirmation, and the user's prompt is shown as usual

#### Scenario: Slow connection
- **WHEN** a connection has not settled when the wait ends
- **THEN** the result reports it as still connecting, and the agent can check it later with the context status tool

### Requirement: UI tools bring the app to the front
Every tool that changes what Fernrohr shows SHALL bring the application to the front and focus the window it acted on. Tools that only read SHALL NOT change focus.

#### Scenario: Open a panel from the terminal
- **WHEN** an agent opens a Pods panel while the user's terminal is the active application
- **THEN** Fernrohr becomes the active application with the window holding that panel focused

#### Scenario: Reads stay in the background
- **WHEN** an agent lists resources or saved layouts while Fernrohr is in the background
- **THEN** Fernrohr stays in the background
