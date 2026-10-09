# Proposal

## Why

An agent can only work with contexts the user has already connected. When it needs one that is
not open, it has to stop and ask the user to connect it, even though Fernrohr knows every
kubeconfig context and already has an add-context flow. Separately, when an agent opens a panel or
loads a layout, the change happens in a window that is usually behind the agent's terminal, so the
user does not see what the agent is showing them.

## What Changes

- Add a `connect_context` MCP tool that connects a kubeconfig context in the frontmost Fernrohr
  window, through the same path as the status bar's add-context control. It asks the user first,
  because connecting widens what the agent can read and may start a tunnel or a credential flow.
- `connect_context` waits a bounded time for the connection to settle, then reports its state:
  connected, waiting for its tunnel, awaiting a manual tunnel's confirmation, or failed.
- Every tool that changes what the UI shows (`open_panel`, `load_layout`, and `connect_context`)
  brings Fernrohr to the front and focuses the window it acted on. Read tools and `list_layouts`
  never take focus. Action approvals already raise their window.

## Capabilities

### New Capabilities

- None.

### Modified Capabilities

- `agent-mcp`: adds the connect-context tool and its approval, and requires UI-changing tools to
  bring the app to the front.

## Impact

- `app/src/mcp/navigate` (new `connect_context` tool, focusing after `open_panel` and
  `load_layout`), `app/src/mcp/approval.rs` (a connect approval request), and
  `util::shell::add_context`.
- The `fernrohr mcp` tool list grows by one tool. No change to the endpoint or adapter.
