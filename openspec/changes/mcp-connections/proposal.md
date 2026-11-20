# Proposal

## Why

When an MCP client connects a context (App#367), Fernrohr adds it to the frontmost window, or opens
a window when none is open. That window is then the connection's only holder, so closing it, or
disconnecting the context from it, drops the connection the agent is still using. The user often
doesn't want a window for a context only an agent uses, and has no way to see which connections
exist or who started them. The approval dialog is right and stays as it is.

## What Changes

- **Connections can be held by an agent, not only by windows.** A connection stays open while any
  window *or* any live MCP session uses it, and closes when the last of them lets go.
- **`connect_context` no longer touches windows.** After the user approves (same dialog as today),
  a context no window holds is connected for the requesting MCP session only, and the new
  Connections window opens (or comes forward) showing it. A context a window already holds behaves
  as today: no approval, no second connection, focus that window.
- **MCP sessions become visible to the app.** The `fernrohr mcp` adapter keeps one session open to
  the app for its whole life, carrying the MCP client's self-reported name from `initialize`.
  Requests without a live session are rejected. When the session ends (the client exits or the
  adapter dies), its holds become ownerless and close after a short grace period; nothing can
  reclaim them, and a restarted client asks for approval again.
- **Approval is bound to the session.** An agent holds a context only when its own session was
  approved for it, or a user window already holds it; it can't ride on another agent's approval,
  and can't read a context only other agents hold. (Security amendments, design.md "Security".)
- **New Connections window**: every connected context across the app, its state and tunnel, and
  who holds it: each window by title, each agent by client name. From it the user can open a
  context in a window, disconnect it entirely, or end one agent's hold. It's a registered command
  (palette, Window menu, status menu), keyboard-first. **Recommended over folding it into the
  Tunnel Manager** (see design.md, decision 1); the context chooser's "Manage Tunnels" wording is
  unchanged either way.
- **Status bar capsule** shows when an agent also holds the window's context, and the capsule's
  Disconnect confirmation says the connection stays open for that agent.
- **Closing the last window** (macOS quiet mode) no longer disconnects agent-held contexts; MCP
  reads keep working in the background. "Stop with the last window" still stops MCP, which ends its
  sessions and their holds.

## Capabilities

### New Capabilities

- `connections-window`: the app-wide Connections window - what it lists, who holds each
  connection, and its open, disconnect and end-agent-hold actions from keyboard and mouse.

### Modified Capabilities

- `cluster-connection`: "Connections shared across windows" - a connection's holders are windows
  and MCP sessions; it stays open while any holds it.
- `agent-mcp`: "Connect context tool" and "Connecting requires approval" - connects for the MCP
  session, approval bound to that session, the Connections window instead of a window; new
  requirements for live sessions, secret session ids, holds ending with the session, reads
  limited to the session's own access, and agent connections surviving the last window.
- `app-shell`: "Disconnect a context from a window" - the confirmation names agents that keep the
  connection open; the status bar capsule marks agent-held contexts.

## Impact

- **App code**: `k8s/cluster/session` (holder type and hold/release), `mcp/` (adapter session,
  protocol `Hello` gains client name and session, `navigate/connect.rs`), `util/shell/agent.rs`
  (`connect_context` no longer opens windows), `util/shell/quiet.rs` (last-window close keeps
  agent holds), `ui/status_bar` (agent marker, disconnect wording), new `ui/connections/` window,
  `command.rs` registrations and status menu item.
- **Protocol**: adapter/app socket protocol version bump; an older adapter is rejected with an
  error telling the user to restart their MCP client.
- **No new dependencies.**
- **docs/site** (`latest/`): `agents.md` (Connect, Approve, When no window is open),
  `reference/mcp-tools.md` (`connect_context`), `clusters.md` (new "See every connection"
  section, Disconnect), `windows-panels.md` (Quiet mode, menus), `tunnels-forwards.md` (pointer to Connections),
  `reference/keybindings.md` (Connections commands).
