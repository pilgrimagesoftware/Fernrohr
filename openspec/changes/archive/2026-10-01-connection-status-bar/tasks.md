# Tasks

## 1. Health model

- [x] 1.1 Record an `Instant` on each `ClusterConnection` state change and add
  `ClusterRegistry::health(cx, context) -> ContextHealth` with precedence failed > paused (first
  paused watch key) > waiting for tunnel > connected. Verify: unit tests for each variant, for
  precedence when a connection is both paused and waiting, and for a pause on a non-`"pods"` key - `k8s/cluster/context_health.rs` (`ContextHealth`, `Severity`), `ClusterRegistry::health` in `session.rs`, `WatchRegistry::first_paused`, and a `since` timestamp on `ClusterConnection`
- [x] 1.2 Add the pure `severity(&ContextHealth, now)` with a `STATUS_ESCALATE_AFTER` (30 s)
  constant in `consts.rs`. Verify: a table test covering every row of the spec's color list,
  including a pause at 29 s (warning) and 31 s (danger).
- [x] 1.3 Audit every pause, resume and connection-state write for a GPUI notify reaching
  `ClusterRegistry` observers, and add any that are missing. Verify: a test pauses and resumes
  through `apply_health_transition` and asserts an `observe_global` callback fires for each edge - every `ClusterRegistry` write already goes through `cx.global_mut`, which notifies global observers (test `pause_and_resume_each_notify_registry_observers`); connection-state changes notify only their own `ClusterConnection` entity, so the bar also observes each shown context's connection

## 2. Status bar view

- [x] 2.1 Add `ui/status_bar.rs`: one item per context the window uses, non-connected first, with
  state icon, context name, text, elapsed time, and the color from `severity`. Verify: GPUI tests
  for two contexts (problem item first), for a closed last panel keeping its item, and for icon and
  text differing per state.
- [x] 2.2 Refresh on `ClusterRegistry` changes, and run a one-second tick only while an item is not
  connected. Verify: a GPUI test with a fake clock showing elapsed time advancing while paused and
  escalating past 30 s, and no timer once all items are connected - elapsed time uses an injected `Clock` because GPUI's test clock uses a different `Instant` type than the pause pipeline; the tick itself runs on the real executor timer and is tested with `advance_clock`
- [x] 2.3 Render the bar under the workspace body in `MainWindow` (column layout, bar fixed height),
  not in picker mode. Verify: a render test that a workspace window contains the bar and a picker
  window does not.

## 3. Remove the panel banner

- [x] 3.1 Remove the "Paused (...)" line from the Pods panel and delete `pods_pause_info` once it
  has no callers; paused panels keep rendering their last rows. Verify: a GPUI test pauses a
  context and asserts the Pods panel shows its rows and no pause text while the status bar shows
  the pause - `pods_pause_info` is removed; `WatchRegistry::pause_info` is now used only by its own tests

## 4. Full verification

- [x] 4.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` all
  pass - 245/245 on four consecutive runs; the keychain tests failed at first only because interrupted earlier runs had left fixed-name test entries in the login keychain (follow-up: give those tests per-run unique ids)
- [x] 4.2 Manual check against the QA bastion: with Pods panels open on two contexts sharing a
  tunnel, kill the forward and hold it down past 30 s (as in `tunnel-bastion-verification` 1.2).
  Confirm each window's status bar shows the context reconnecting in the warning color, turns to
  danger after 30 s, returns to muted on resume, and that no panel shows its own pause line.
 - verified by Paul in the running app (2026-09-29)