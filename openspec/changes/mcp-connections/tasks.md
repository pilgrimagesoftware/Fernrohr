# Tasks

## 1. Holder model

- [ ] 1.1 Replace `ClusterSession.holders: HashSet<WindowId>` with a set of `Holder::{Window, Agent}` and record the creating holder; move every existing caller to `Holder::Window`. Verify the existing hold tests (`closing_a_window_releases_only_its_own_hold` and the `holds.rs` tests) pass unchanged.
- [ ] 1.2 Add `release_agent(session)` and a read of a context's holders. Verify with unit tests: a window and an agent sharing a context, the window closing leaves it connected, the agent releasing last drops the session (cluster-connection "Window closes while an agent holds the context", "Agent's session ends last").
- [ ] 1.3 Route `ClusterRegistry::retry_with` through `ClusterConnection::redial` (App#431) instead of replacing a session no window holds, and store `context_name` on `ClusterConnection` so `redial` reads its own. Verify with a test that retrying a Failed context held only by an agent reconnects the same entity and the agent's hold sees `Connected` (cluster-connection "Retry keeps an agent's hold").
- [ ] 1.4 Write the App ADR recording that connections are held by windows and agent sessions (design decision 2) in the App's `docs/adr/`, and update `docs/architecture.md`'s session description. Verify the ADR index lists it.

## 2. MCP sessions

- [ ] 2.1 Extend the adapter/app protocol: bump the version, add `OpenSession{client_name}` answered with a session id, and a session id on each request. Verify with protocol round-trip tests, and that a request with no, unknown or ended session id is rejected with the "restart your MCP client" error before any tool runs (agent-mcp "Older adapter", "Ended session").
- [ ] 2.2 Have the adapter open the session socket on its first request (client name from `initialize`'s `clientInfo`) and keep it for its lifetime. Verify with an adapter test that tool calls carry the session id and that the socket stays open between calls.
- [ ] 2.3 In the app, track live sessions; on a session socket closing, turn its holds into `Holder::Lingering` and release them after the grace period (`consts.rs`). Verify with tests: the connection stays open during the grace period, no session can claim a lingering hold, a restarted client is asked for approval again, and the hold is released after the period (agent-mcp "Client exits", "Client restarts within the grace period").
- [ ] 2.4 Generate session ids as 128 bits from `getrandom`, compare them in constant time, and keep them out of logs, UI and tool results. Verify with a test that captures debug logs over a session's life and finds no id (agent-mcp "Identifier stays out of logs"), and one that an ended id is never accepted again.
- [ ] 2.5 Stopping MCP (setting or `StopWithLastWindow`) ends every session. Verify with a test that stopping releases agent holds (agent-mcp "Stop serving with the last window").

## 3. `connect_context` connects for the session

- [ ] 3.1 Change the connect path to follow design decision 4's table: hold for the requesting session with no window change; no approval only when the session already holds it or a user window does; the same approval dialog when only other sessions or a lingering hold have it. Verify with MCP connect tests for each agent-mcp "Connect context tool" and "Connecting requires approval" scenario (including "Context only another agent holds" and "Denied while another agent holds it"), and that the approval dialog's content is unchanged.
- [ ] 3.2 Make read tools session-aware: answer only for contexts a user window or the requesting session holds, else report not connected. Verify with tests for agent-mcp "Another agent's connection" and "Window-held context".
- [ ] 3.3 Keep client names display-only: store the self-reported name per session and label it as self-reported wherever shown. Verify with a test that two sessions reporting the same name get separate holds and separate approval.
- [ ] 3.4 Update `docs/site/latest/reference/mcp-tools.md` (`connect_context`) and `agents.md` ("Connect" and "Approve actions and connections"), covering per-agent approval, the restart error and the self-reported names, with placeholder context names. Verify the docs build and link checks pass.
- [ ] 3.5 Make `open_panel` on a context only the calling session holds add it to the frontmost window, or open a window on it, which then holds it too (design decision 4). Verify with a test that the panel opens and both the window and the agent are listed as holders.

## 4. Connections window

- [ ] 4.1 Add the singleton Connections window (`ui/connections/`): a live list of contexts with at least one holder, showing state, tunnel and holders by window title and agent name. Verify with window tests for "Agent-only connection is listed", "Connection closes", "Nothing connected" and "Window and agent share a connection".
- [ ] 4.2 Register the Connections command (palette, Window menu, status menu, default key checked against `keymap::conflicts`) and open/focus the window from `connect_context` with the new row selected. Verify with a keystroke test that opens it and a connect test that brings it forward.
- [ ] 4.3 Add Open in Window, Disconnect (recoverable confirmation naming windows and agents) and End Agent's Hold as window-scoped commands with icon buttons, tooltips and a hint row; the selection moves only by arrows and clicks. Verify with keystroke tests for each action and a click test for each button (connections-window scenarios).
- [ ] 4.4 Document the window in `docs/site/latest/clusters.md` (new "See every connection" section), list its commands in `reference/keybindings.md`, add a pointer in `tunnels-forwards.md` and the Window and status menu entries in `windows-panels.md`. Verify the docs build and link checks pass.

## 5. Status bar

- [ ] 5.1 Show the agent icon with the agents' names in the tooltip on capsules whose context an agent holds. Verify with a status bar test for "Agent shares a window's context" and "Agent lets go", asserting the tooltip text.
- [ ] 5.2 Include agents in the capsule's Disconnect confirmation. Verify with a test for "Context an agent still uses".
- [ ] 5.3 Update `docs/site/latest/clusters.md` ("Watch the connection", "Disconnect") for the agent icon and wording. Verify the docs build.

## 6. Last window and quiet mode

- [ ] 6.1 Make quiet mode keep agent holds (only window holds are released on close) and keep port-forwards stopping as today. Verify with a test that a read after closing every window succeeds (agent-mcp "Read after closing every window").
- [ ] 6.2 Update `docs/site/latest/agents.md` ("When no window is open") and `windows-panels.md` ("Quiet mode"). Verify the docs build.

## 7. Integration

- [ ] 7.1 Run the App gate (`cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, `cargo test`) and an end-to-end check with a real MCP client against a local cluster: connect, close every window, read, exit the client, and see the connection leave the Connections window after the grace period.

## Workflow follow-up

- Archive the change after the App PRs merge and the user's review is satisfied.
