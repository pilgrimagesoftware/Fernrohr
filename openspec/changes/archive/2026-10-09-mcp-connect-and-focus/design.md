# Design

## Context

The MCP navigation tools (`open_panel`, `list_layouts`, `load_layout`) run on Tokio and cross to the
GPUI main thread through `Foreground`. They act on the frontmost main window holding the request's
context and never connect anything. Action tools go through the approval gate in `mcp/approval.rs`,
whose dialog raises its window. `MainWindow::add_context` (in `util::shell::contexts`) adds a context
to a window, reusing an open connection, opens a Pods panel, and selects it. `notify::focus` already
activates the app and a window. See [proposal.md](proposal.md).

## Goals / Non-Goals

**Goals:**

- Let an agent bring a context online without the user leaving the agent, with the user still in
  control of which clusters it can reach.
- Make the user see every UI change an agent makes, as it happens.

**Non-Goals:**

- Disconnecting contexts from MCP, or connecting without approval.
- Raising the app for read tools, or for `list_layouts`.

## Decisions

### D1. `connect_context` reuses the add-context path

The tool resolves the name against the kubeconfig (an unknown name is `unknown_context`), then on
the main thread picks the frontmost workspace window. If that window already holds the context, it
focuses it and reports its state with no approval. Otherwise it asks for approval, then calls
`MainWindow::add_context`, so tunnel binding, connection sharing, the Pods panel, and selection all
behave exactly as the status bar's add-context control. With no workspace window open, it opens one
for the context, as the picker does. It is a `Navigate` tool that passes through the approval gate.
The allowlist test, which only covers `Action` tools, is unchanged.

### D2. Approval is the recoverable tier

Connecting reveals a cluster to the agent and may start a tunnel, a manual tunnel prompt, or an
exec credential plugin, so the user approves it. It changes nothing in a cluster and is undone by
disconnecting, so the dialog uses the recoverable tier (Enter approves). It names the context, its
bound tunnel and that tunnel's kind, and notes that the agent will be able to read it. It shares the
approval gate's one-question-at-a-time rule and timeout.

### D3. Bounded settle wait

After `add_context`, the tool watches the context's health for up to 20 seconds and returns the
first settled state: `connected`, `waiting_for_tunnel`, `awaiting_confirmation`, `failed` (with
`ToolError::from_kube`-style redaction), or `connecting` when the wait ends. Awaiting confirmation is
returned at once, since only the user can end it. The connection continues after the tool returns;
`list_contexts` reports later changes.

### D4. Bring to front after UI changes

A shared `focus_window(handle, cx)` helper does what `notify::focus` does: `cx.activate(true)`, then
`window.activate_window()`. `open_panel` and `load_layout` call it on the window they acted on, after
the change, in the same foreground task. `connect_context` calls it on the window it added to or
opened. The approval dialog keeps raising its own window. `list_layouts` and every read tool never
call it.

### D5. Implementation notes

- The approval request's kind is now `Asking::Action { irreversible }` or `Asking::Connect`, so a
  connection cannot be marked irreversible. The connect dialog has its own wording and no
  namespace or kind rows.
- `ToolContext` carries the tunnels file path (for the tunnel name in the question) and the
  settle wait (`MCP_CONNECT_SETTLE`, 20 seconds, polled every 100 ms).
- `paused` also counts as settled: the connection is up and its watches are waiting.
- `focus_window` lives in `util::shell::agent`, and `notify::focus` now calls it.

## Risks / Trade-offs

- [Focus stealing while the user types elsewhere] -> Only tools whose purpose is to show the user
  something take focus, and only once per call.
- [An approved connection runs an exec credential plugin that opens a browser] -> The approval
  dialog names the context, so the user knows a sign-in may follow. This is the same flow as
  connecting by hand.
- [A long connect outlives the tool call] -> The tool reports `connecting`, and the connection
  carries on in the app as usual.
