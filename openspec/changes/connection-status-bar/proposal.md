# Proposal

## Why

When a tunnel drops or credentials refresh, the only sign is a plain-text "Paused (...)" line
inside each affected Pods panel. It repeats once per panel for a single outage, has no color, and
is easy to miss. It showed up that way in the 2026-09-29 QA bastion flap test
(`tunnel-bastion-verification` 1.2). Connection trouble is window-level news and needs one place
that catches the eye.

## What Changes

- A **status bar** along the bottom of every workspace window, with one item per cluster that has
  panels in that window: context name, state, and elapsed time for any non-healthy state.
- **Colors by severity**: connected is muted, waiting for tunnel and credential refresh use the
  info color, tunnel reconnecting uses the warning color, and a failed connection, or a pause that
  has lasted past a threshold, uses the danger color. Every state also has a distinct icon and
  text, so the bar never relies on color alone.
- Non-healthy items sort before healthy ones, and elapsed time ticks while a pause lasts.
- **Removed**: the per-panel "Paused (...)" line in the Pods panel. Panels keep showing their last
  data while paused.

Not in scope: tunnel management (`tunnel-management-ui`), a notification or toast system, and
status for the cluster picker view, which already shows its own connect progress.

## Capabilities

### New Capabilities

None.

### Modified Capabilities

- `app-shell`: workspace windows gain a status bar summarizing each cluster's connection state.
- `cluster-connection`: the paused state of a recoverable interruption is shown in the window's
  status bar with its reason and elapsed time, not inside each panel.

## Impact

- App code: a new `ui/status_bar.rs` rendered by `MainWindow` in workspace mode (`util/shell.rs` is
  already far past the file-size cap, so the bar lives in its own module), a per-context health
  query on `ClusterRegistry` (`k8s/cluster/session.rs`) combining connection state and pause info,
  and removal of the banner in `k8s/resource/pods.rs`.
- Theme: uses the existing semantic colors (`success`/`info`/`warning`/`danger`) from
  gpui-component. No new theme tokens.
- No new dependencies. No config changes.
