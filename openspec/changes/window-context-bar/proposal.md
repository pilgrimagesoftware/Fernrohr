# Proposal

## Why

A workspace window is locked to the one context picked when it opened. The only way to see a
second cluster is a second window, and the only way to drop a cluster is to close panels one by
one. Side-by-side clusters in one dock is the project's stated differentiator. The Resource panel's
cluster dropdown and `MainWindow`'s `connection_count` were built for it, but nothing adds a second
connection to a window yet.

## What Changes

- A **context bar** along the top of every workspace window, under the title bar, with one chip per
  context the window uses. Each chip shows the context name, its tunnel if bound, and a small
  health dot using the same severity as `connection-status-bar`. The bottom status bar keeps the
  detail: reason and elapsed time.
- **Add context** ("+"): a picker popover listing the contexts this window doesn't use yet, with
  their tunnel bindings. Picking one connects it, or reuses the existing connection if another
  window already has it, then opens its Pods panel and switches the Resource panel's cluster
  dropdown to it.
- **Disconnect context** (chip menu): after a confirmation naming how many panels will close, it
  closes every panel in this window that uses the context and removes the chip. If other windows
  still use the context, its connection stays up for them, and the confirmation says so. The
  connection, and its tunnel, shut down only when no window uses it.
- Disconnecting a window's last context returns that window to the cluster picker.
- Workspace persistence records the list of contexts per window and restores them all.

Not in scope: renaming or reordering chips, moving a panel between contexts, and per-chip tunnel
controls (tunnels are managed in `tunnel-management-ui`).

## Capabilities

### New Capabilities

None.

### Modified Capabilities

- `app-shell`: workspace windows gain a context bar with add and disconnect; workspace persistence
  restores every context a window used.
- `cluster-connection`: one connection per context is shared across windows and shut down only when
  the last window using it disconnects it or closes.

## Impact

- App code:
  - `util/shell.rs`: `WindowMode::Workspace` holds a list of contexts instead of one
    `context_name`. The bar itself goes in a new `ui/context_bar.rs`, since `shell.rs` is far past
    the file-size cap.
  - `k8s/cluster/session.rs`: `ClusterRegistry` gains per-window holds and releases the session on
    the last one.
  - The Resource panel's cluster dropdown and the title bar's connection count become live.
  - `config/` workspace state stores a context list per window, with old single-context files still
    loading.
- Depends on `connection-status-bar` for `ClusterRegistry::health` and `severity()`, and shows
  tunnel names from `tunnel-management-ui` when that has landed. The chip simply omits the tunnel
  name if it hasn't.
- No new dependencies.
