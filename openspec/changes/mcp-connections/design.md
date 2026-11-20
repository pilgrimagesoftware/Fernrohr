# Design

## Context

See proposal.md for why. The current code (App `develop`):

- **Holders are windows only.** `ClusterSession.holders` is a `HashSet<WindowId>`
  (`k8s/cluster/session/registry.rs`); `holds.rs` `hold`/`release`/`release_window` drop the session,
  and with it the connection, watches and tunnel forward, when the set empties. The app-wide
  `on_window_closed` releases every hold a window had.
- **MCP holds nothing.** MCP reads use `existing_connection`, which never connects or holds.
  `connect_context` (`mcp/navigate/connect.rs`) approves, then `shell::connect_context`
  (`util/shell/agent.rs`) adds the context to the frontmost workspace window, or opens a window when
  none exists. That window becomes the only holder.
- **There is no MCP session in the app.** `fernrohr mcp` (`mcp/adapter.rs`) opens a fresh Unix
  socket per tool call: `Hello{protocol, token}`, one `Request`, one `Reply`, close. No client name
  or session id reaches the app, though the adapter sees `initialize`'s `clientInfo`.
- **Quiet mode** (macOS, `util/shell/quiet.rs`): when `cx.windows()` empties, the app stays
  running, stops port-forwards, saves the workspace, and stops MCP only under
  `BackgroundMcp::StopWithLastWindow`. By then every session has been released, so nothing is
  connected. Elsewhere the app quits with its last window.
- **Tunnel Manager** (`ui/tunnels/list/`): a singleton window (720x560) stacking tunnels, running
  port-forwards and stale bindings, opened by "Manage Tunnels…" (`tunnels.manage`), the context
  chooser's "Manage tunnels…" button and the status menu's "Tunnel Manager…".

## Goals / Non-Goals

**Goals:**
- A connection's lifetime is the union of its window and agent holders.
- The app knows each MCP client as a session with a name, and when it ends.
- One place lists every connection and who holds it, operable from the keyboard.

**Non-Goals:**
- Changing the approval dialog, its timeout, or which tools need approval.
- Keeping the app alive after the last window on Linux and Windows (it quits there today, ending
  MCP with it); that is a separate platform decision.
- Port-forwards held by agents. Forwards still belong to panels and still stop in quiet mode.
- A `disconnect_context` MCP tool. The user ends holds in the Connections window; a tool can follow.

## Decisions

### 1. A separate Connections window, not a section of the Tunnel Manager

Recommended: a new singleton **Connections** window. Folding it into the Tunnel Manager was
considered and rejected:

- **It's runtime state, not configuration.** The Tunnel Manager edits saved tunnel definitions;
  Connections lists live sessions across every context, most with no tunnel at all. A context on a
  direct connection showing up under "Tunnel Manager" would read wrong.
- **`connect_context` opens it automatically.** Popping the Tunnel Manager every time an agent
  connects would put tunnel editing in front of the user for an unrelated event, and its title and
  entry points ("Manage Tunnels…", "Tunnel Manager…") must keep their wording (App#367).
- **Room and focus.** The Tunnel Manager is already a full stack of three sections at 560px. A
  fourth list with its own selection and commands would compete for arrow keys and the hint row.
- **Cost is small.** The singleton-window pattern (`TunnelsWindowHandle`) and the port-forwards
  section's row layout are reused; the Tunnel Manager itself doesn't change.

Trade-off: one more window and command. Accepted; it's reachable from the palette, the Window menu
and the status menu.

### 2. Holder model: windows and agent sessions

`holders` becomes a set of `Holder::{Window(WindowId), Agent(AgentSessionId)}`, with
`hold`/`release` taking a `Holder` and `release_agent(session)` mirroring `release_window`. The
session records the holder that created it, so the Connections window can say who connected it.
Every existing caller passes `Holder::Window`, so window behaviour is unchanged.

Alternative: a separate refcount for agents beside the window set. Rejected: two counters to keep
in step, and "who holds it" needs identities anyway.

**Decision record candidate**: connection ownership moves from "windows" to "windows and agent
sessions". It's a structural change to the session model the App's `docs/architecture.md` describes;
an `ADR-*` in the App's `docs/adr/` should record it at implementation time.

### 3. One long-lived MCP session per adapter

The adapter opens a **session socket** when it receives `initialize` and keeps it for its whole
life: `Hello{protocol, token}` then `OpenSession{client_name}`, answered with a session id. Each
per-call request carries that id. The app treats the session socket closing (client exit, adapter
crash, MCP stopped) as the session ending and releases its holds after a short grace period
(a `consts.rs` decision, 30 s), so a client that restarts and reconnects within it finds the
context still connected and needs no fresh approval.

Alternatives:
- *Idle timeout on agent holds.* Rejected: arbitrary, and drops a connection an agent is still
  about to use.
- *Release only by the user.* Rejected: a crashed agent would hold a cluster open until noticed.

**Compatibility**: the protocol version goes up. A request with no session id (an older adapter)
is attributed to one shared "Unknown agent" session that lives until MCP stops or the user ends it
in the Connections window.

### 4. `connect_context` connects for the session and shows the Connections window

After approval (unchanged), the context is held by `Holder::Agent(session)` and no window is
touched. The Connections window opens or comes forward, selecting that row: that is the "UI tools
bring the app to the front" focus for this tool. A context that's already connected takes the
agent hold without approval; if the frontmost window holds it, that window is focused as today.
`open_panel` on an agent-only context still opens a window, which then holds it as well.

### 5. Last window and quiet mode

Closing a window releases only its `Window` holds, so agent-held contexts survive quiet mode, and
reads keep answering. `enter` still stops port-forwards. `StopWithLastWindow` stopping MCP closes
every session socket, which ends their holds through decision 3. On Linux and Windows the app quits
as before. The Connections window counts as an open window, like the Tunnel Manager, so quiet mode
starts only once it closes too.

### 6. Retrying a failed connection keeps every holder

Built on App#431's `ClusterConnection::redial`, which reconnects the same connection entity in
place, so every holder (windows and agent sessions) sees the retry without re-subscribing. Two
changes keep that true once agents hold connections (from the #431 review):

- **The picker's retry goes through `redial` too.** `ClusterRegistry::retry_with` currently removes
  a session no window holds and builds a new entity. With agent holders, "no window holds it" no
  longer means "nobody observes it", and the swap would leave an agent on the orphaned, Failed
  entity. The swap goes; retry always redials in place.
- **`ClusterConnection` stores its own `context_name`**, and `redial` reads it rather than taking
  it from the caller, so a connection more owners can reach can't be redialled at another context.

Ownership stays on `ClusterSession` (decision 2 only widens its holder set), so the session's
`_health` task, which drives pause and resume from the tunnel, keeps serving agent-only holders
with no change.

This change is based on `develop` after App#431 merges.

### 7. Status bar

A capsule whose context any agent holds shows an agent icon from the app's existing icon set, with
a tooltip naming the agents. The capsule's Disconnect confirmation counts agents alongside other
windows ("stays connected for assistant"). Agent-only contexts have no capsule anywhere; the
Connections window is where they're seen.

## Risks / Trade-offs

- [A session socket that dies silently (sleep, suspended process)] → closure is detected when the
  OS reports it; the user can always end a hold by hand in the Connections window.
- [Agents keeping clusters connected the user forgot about] → the Connections window opens on every
  new agent connection, and the capsule's agent icon shows shared ones.
- [Protocol change between adapter and app] → version bump with the "Unknown agent" fallback;
  adapter and app ship in one binary, so mismatches only arise across a running upgrade.
- [Holder-less sessions created lazily by `ClusterRegistry::connection`] → the Connections window
  lists only contexts with at least one holder, so those transient sessions don't appear.

## Migration Plan

No persisted state changes. Ships in one release; rollback is reverting the release. The
workspace file still records each window's contexts, so restored windows behave as before.
