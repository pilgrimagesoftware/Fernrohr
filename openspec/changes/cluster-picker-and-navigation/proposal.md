# Proposal

## Why

The app currently has no front door: `shell::open_window` hardcodes a fixed
two-panel dock (Pods + Logs) with no cluster selection and no way to switch
resource kinds. `cluster-connection`'s "connect to one user-selected context"
requirement was never given a UI, and `resource-browser` only ever shows
Pods. A user cannot actually launch the app, pick a cluster, and look around
- which is the app's basic reason to exist.

## What Changes

- Add a startup screen shown when a window has no restored panels: lists
  kubeconfig contexts (via the existing `cluster::kubeconfig` module),
  lets the user pick one, and drives `cluster::connection` to connect.
- On successful connection, replace the picker with the window's panel
  workspace, landing on a default resource view for that cluster instead of
  a hardcoded Pods+Logs split.
- Add resource-kind navigation (a sidebar) so an open panel can switch which
  resource kind it displays, backed by the discovery data `cluster-connection`
  already exposes, instead of one kind being wired in at panel-construction
  time.
- Surface a connection failure (bad context, unreachable cluster) on the
  picker itself rather than failing silently into a blank window.
- **BREAKING**: `shell::open_window`'s hardcoded panel construction is
  removed; a window with no saved layout now opens the picker, not
  Pods+Logs.

## Capabilities

### New Capabilities
- `cluster-picker`: the startup/context-selection screen and its connect flow.

### Modified Capabilities
- `app-shell`: what a window shows before any panel is open or restored
  (the picker) instead of nothing being specified.
- `resource-browser`: panels can switch resource kind via navigation instead
  of being fixed to Pods at construction.

## Impact

- `app/src/shell.rs`: `open_window` no longer hardcodes panel construction;
  branches on whether the restored layout has panels.
- `app/src/cluster/`: `kubeconfig.rs` (list contexts) and `connection.rs`
  (connect) get a UI-facing entry point; `discovery.rs` output needs to reach
  the new sidebar.
- New UI module for the picker view and the resource-kind sidebar.
- `resource_index.rs` / `pods.rs`: panel construction decouples from "always
  Pods" to "whatever kind is selected."
