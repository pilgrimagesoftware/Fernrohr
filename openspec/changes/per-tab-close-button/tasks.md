# Tasks

## 1. Decision

- [x] 1.1 Option C chosen (2026-09-30) - see `proposal.md`. Revisit Option A separately once the
  `gpui-component` 0.7.0+ changelog can actually be checked.

## 2. Cmd-W: close active tab, else close window

- [x] 2.1 Added `MainWindow::has_open_panel` (reuses `DockArea::is_empty(DockPlacement::Center,
  cx)`, already used by `watch_workspace`).
- [x] 2.2 `close_tab_or_window` (`util/shell.rs`) dispatches `ClosePanel` when the dock has an
  open panel, else falls through to the window-close path. `CloseWindow`'s handler
  (`ui/menu.rs::init`) now calls it instead of `close_window` directly - same single registered
  command/binding, only the handler's behavior changed.
- [x] 2.3 Added `ClusterRegistry::has_active_tunnel` (`k8s/cluster/session.rs`), backed by a pure
  `is_tunnel_active` helper (`Up`/`Reconnecting` count as active).
- [x] 2.4 Added `context_lifecycle::close_window_confirmation_body` and `confirm_close_window`
  (`util/shell.rs`), modeled on `context_bar::open_disconnect_dialog` /
  `disconnect_confirmation_body`. Empty `active_tunnels` skips the dialog and closes immediately.
- [x] 2.5 Unit tests cover the decision logic directly (`is_tunnel_active`, `has_active_tunnel`'s
  false paths, `close_window_confirmation_body`'s wording) and `close_tab_or_window`'s
  window-close branch end-to-end. The `ClosePanel`-dispatch branch and the confirm-dialog branch
  are not covered by an automated test - see the doc comment on
  `cmd_w_routing_closes_the_window_when_no_panel_is_open_and_no_tunnel_is_active`
  (`util/shell/tests.rs`) for why (needs a painted, focused render tree this codebase's test
  harness has no precedent for; a real, reachable SSH bastion for `ForwardState::Up`). Verified
  manually instead: the app builds, launches, and runs without crashing; `cargo clippy -D
  warnings`, `cargo fmt --check`, and the full `cargo test` suite (401 tests) are clean.
- [x] 2.6 Menu bar's "Close Window" item and `Cmd-W` share one registered command
  (`window.close`), so there is no second binding to drift - already proven by
  `platform_items_have_their_standard_shortcuts` (`ui/menu.rs`).
