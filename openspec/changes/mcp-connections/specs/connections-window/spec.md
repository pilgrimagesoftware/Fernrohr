# Spec Delta

## Purpose

Shows every cluster connection open in Fernrohr, whichever window or agent holds it, and lets the
user open, disconnect or release those connections without needing a window per context.

## ADDED Requirements

### Requirement: Connections window lists every connection
The application SHALL provide a single Connections window listing every context with an open
connection, whether a window, an MCP session, or both hold it. Each row SHALL show the context
name, its connection state with the status bar's icon and state text, its bound tunnel if any, and
its holders. The list SHALL update live as connections open, change state, or close.

#### Scenario: Agent-only connection is listed
- **WHEN** an agent's approved connection to `cluster-a` is held by no window
- **THEN** the Connections window lists `cluster-a` with its state and the agent as its only holder

#### Scenario: Connection closes
- **WHEN** the last holder of `cluster-b` lets go
- **THEN** `cluster-b`'s row disappears from the Connections window

#### Scenario: Nothing connected
- **WHEN** no context is connected
- **THEN** the window says that no context is connected

### Requirement: Holders identify who uses a connection
Each row SHALL name every holder: each window by its title, and each MCP session by the name its
client reported, labelled as self-reported ("reports itself as …"), or as an unnamed agent when it
reported none. A connection an agent started SHALL say so even after a window also takes it.

#### Scenario: Window and agent share a connection
- **WHEN** a window and an agent both hold `cluster-a`, and the agent started it
- **THEN** the row lists the window's title and the agent as "reports itself as" its name, and marks the agent as the one that connected it

#### Scenario: Client without a name
- **WHEN** an MCP client that reported no name holds `cluster-c`
- **THEN** the row lists it as an unnamed agent

### Requirement: Open a connection in a window
The Connections window SHALL offer, for the selected row, an action that adds the context to the
frontmost workspace window, or opens a window for it when none is open. The window SHALL then hold
the context without a new connection or approval, and SHALL take focus.

#### Scenario: Open an agent's connection
- **WHEN** the user opens `cluster-a`, held only by an agent, from the Connections window
- **THEN** the frontmost window gains `cluster-a` and its Pods panel, the connection is reused, and the row lists that window as a holder too

### Requirement: Disconnect a connection from the Connections window
The Connections window SHALL offer, for the selected row, a Disconnect action that releases every
holder of that context. Before acting it SHALL ask for confirmation naming each window that will
close the context's panels and each agent that will lose it. Disconnect is a recoverable
confirmation that Enter may confirm and Escape cancels.

#### Scenario: Disconnect a shared connection
- **WHEN** the user disconnects `cluster-a`, held by one window and one agent, and confirms
- **THEN** the window closes its `cluster-a` panels and capsule, the agent's hold ends, and the connection closes

#### Scenario: Cancel
- **WHEN** the user cancels the confirmation
- **THEN** every holder keeps `cluster-a` and the connection stays open

### Requirement: End one agent's hold
The Connections window SHALL offer, for an agent holder of the selected row, an action that ends
only that agent's hold, without a confirmation. If no other holder remains, the connection SHALL
close. The agent's later requests for that context SHALL behave as if it had never connected it.

#### Scenario: Release an agent that's done
- **WHEN** the user ends the agent's hold on `cluster-a`, which no window holds
- **THEN** `cluster-a` disconnects and the agent's next read of it reports it isn't connected

#### Scenario: Re-connecting after release needs approval
- **WHEN** the user has ended an agent's hold on `cluster-a` and that agent asks to connect `cluster-a` again
- **THEN** the user is asked to approve, as for a new connection

### Requirement: Opening the Connections window
The Connections window SHALL be opened or brought forward by a registered command, offered in the
command palette, the Window menu and the status menu, with a default key the user's keymap can
override. The context chooser's "Manage Tunnels" item and the Tunnel Manager SHALL be unchanged.

#### Scenario: Open from the palette
- **WHEN** the user runs Connections from the command palette
- **THEN** the Connections window opens, or comes forward if it is already open, with its list focused

### Requirement: Connections window is keyboard operable
The Connections window's selection SHALL move with the arrow keys and with clicks, never with
hover. Open in Window, Disconnect and End Agent's Hold SHALL be registered commands scoped to the
window, shown as icon buttons with tooltips and in a hint row with their keys.

#### Scenario: Disconnect from the keyboard
- **WHEN** the user selects `cluster-a` with the arrow keys and presses Disconnect's key, then Enter
- **THEN** `cluster-a` disconnects as if the button had been clicked and confirmed
