# Design

## Context

See proposal.md for motivation. Current state in the App repo:

- Pause state lives in each `ClusterSession`'s `WatchRegistry` as `(PauseReason, Instant)` per
  watch key. `ClusterRegistry::pods_pause_info` exposes only the `"pods"` key, and the Pods panel
  (`k8s/resource/pods.rs`) renders it as a plain `div` line.
- `ConnectionState` is `Connecting | WaitingForTunnel | Connected | Failed(String)`.
- `MainWindow` (`util/shell.rs`, 1889 lines) renders the picker or the workspace body. In workspace
  mode it already records every panel it opened, and each panel's `PanelScope` carries its
  `context_name`.
- gpui-component's theme provides `success`/`info`/`warning`/`danger` and muted foreground colors,
  and an icon set (already used for badges in `pod_detail.rs`).

## Goals / Non-Goals

**Goals:**
- One status surface per window, driven by one per-context health query.
- A single source of truth for "is this context healthy", reusable by later features such as tab
  badges and a menu-bar indicator.

**Non-Goals:**
- Toasts, sounds, or OS notifications.
- Showing tunnel-level detail (which bastion, forward count). That belongs to the Tunnels window in
  `tunnel-management-ui`.

## Decisions

### 1. `ClusterRegistry::health(cx, context) -> ContextHealth`

This returns one enum: `Connected`, `WaitingForTunnel { since }`, `Paused { reason, since }`, or
`Failed { reason, since }`. It combines the session's `ConnectionState` with the **first paused
watch key**, not just `"pods"`, so kinds added later are covered automatically. `pods_pause_info`
is removed once the Pods panel no longer calls it.

To get `since` for the waiting and failed states, `ClusterConnection` records an `Instant` on each
state change. Paused already has its `Instant`.

*Alternative:* have the bar read `WatchRegistry` and `ConnectionState` directly. It's rejected
because it would copy the precedence rules (failed > paused > waiting > connected) into UI code.

### 2. Severity is a pure function

`severity(&ContextHealth, now) -> Severity { Muted, Info, Warning, Danger }` is a pure function,
with the 30-second escalation as a named constant in `consts.rs`. `ui/status_bar.rs` maps
`Severity` to theme colors and `ContextHealth` to icon and text. That keeps the spec's color table
testable without rendering.

### 3. Contexts come from the window's own context list

The bar lists the contexts the window uses. Today that is the workspace's single `context_name`.
`window-context-bar` replaces it with an explicit list, which the bar reads unchanged. Closing
panels never removes an item; only removing the context from the window does.

### 4. Refresh: observe the registry, tick only while unhealthy

The status bar view observes the `ClusterRegistry` global for state changes. While any item it
shows is not `Connected`, it also runs a one-second timer to advance elapsed times and cross the
escalation threshold, and it stops the timer once everything is connected. *Alternative:* a
permanent one-second tick. It's rejected because it wakes an idle app every second.

The implementation must confirm that every pause, resume and connection-state write goes through
`cx.update_global`, or add a notify. Otherwise `observe_global` misses edges. That check is its own
task.

### 5. Placement and layout

`MainWindow`'s workspace branch becomes a column: the existing row (resource panel and dock) with
`flex_1`, then the status bar at a fixed small height. The picker view has no bar. Items are left
aligned, with state icon, context name, text and elapsed time. The bar uses the UI font, matching
the `pods-table-polish` direction away from monospace outside logs.

## Risks / Trade-offs

- [`observe_global` misses edges if a write path mutates the registry without notifying] → Covered
  by decision 4's audit task and a test that pauses through the real path and asserts a re-render.
- [30-second escalation feels arbitrary] → It's a single constant, easy to tune after use. The
  spec pins the behavior, not the tuning rationale.
- [Removing the panel banner loses context when a window's status bar is off-screen] → The bar is
  part of every workspace window, so each window showing a paused context has its own bar.
