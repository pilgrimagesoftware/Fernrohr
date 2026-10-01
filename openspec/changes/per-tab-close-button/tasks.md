# Tasks

## 1. Decision

- [x] 1.1 Option C chosen (2026-09-30) - see `proposal.md`. Revisit Option A separately once the
  `gpui-component` 0.7.0+ changelog can actually be checked.

## 2. Cmd-W: close active tab, else close window

- [x] 2.1-2.6 Implemented independently by another session, on the `App` repo's `develop`
  directly (commit `9949805`, `feat(ui): switch tabs from the keyboard; Cmd-W closes the focused
  tab`), concurrently with - and before - this session's own attempt at the same tasks. That
  implementation is in `app/src/util/shell/tabs.rs` (`MainWindow::on_action_close_window`,
  `losing_a_tunnel`, `open_close_window_dialog`/`close_window_confirmation_body`) and is more
  correct than this session's draft: it focuses the dock's active tab before dispatching
  `ClosePanel` (so `Cmd-W` closes something with focus elsewhere in the window too), and its
  tunnel-teardown check (`losing_a_tunnel`) only warns when this window is the *last* holder of a
  context, not merely when a tunnel happens to be active (which would warn even when another
  window keeps it alive). This session's own version was discarded unmerged - see
  `pilgrimagesoftware/Fernrohr-App#51` (closed, superseded).
