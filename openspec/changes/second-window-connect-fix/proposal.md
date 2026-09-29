# Proposal

## Why

Opening a second window and selecting a cluster context in its picker reports that the
connection succeeded, but the window never leaves picker mode to show the workspace - a
regression against the `app-shell` capability's existing "each window gets an independent panel
workspace" requirement, observed during manual testing after the cluster-picker-and-navigation
change landed. This is a pure bugfix: no new behavior is being introduced, the already-specified
behavior is broken for the second-window case specifically (a single window works correctly).

## What Changes

- Diagnose why `MainWindow::enter_workspace` (or the `PickerEvent::Connected` subscription that
  triggers it) does not run, or does not take visible effect, for a window other than the first
  one opened.
- Fix whatever the root cause turns out to be, with a regression test that specifically opens two
  windows (not one) and drives the second one's picker to a connected state.

## Capabilities

### New Capabilities
(none)

### Modified Capabilities
(none - the existing `app-shell` requirement already covers this; behavior is not changing, a
bug is. See `skip_specs: true` in `.openspec.yaml`.)

## Impact

- `app/src/util/shell.rs`: `watch_picker`, `enter_workspace`, and `open_window` are the prime
  suspects - this is where per-window state is assembled and where the `PickerEvent::Connected`
  subscription is wired.
- `app/src/ui/picker.rs`: `ClusterPicker::select`/`emit_connected` is the other side of that
  subscription; worth confirming it fires correctly when a *second* window's picker selects a
  context that a first window has already connected (registry-shared `ClusterConnection` reused
  vs freshly created affects whether `cx.observe`'s callback ever fires, since the entity may
  already be in its terminal `Connected` state before the second window's `cx.observe` attaches -
  see design.md).
- No persistence/config format changes anticipated.
