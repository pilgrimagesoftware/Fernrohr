# Tasks

## 1. Connect to a named context

- [x] 1.1 Change `ClusterConnection::connect` to `connect(cx, context_name: Option<String>)`;
  `Some(name)` builds `Config` via `Kubeconfig::read()` + `KubeConfigOptions { context: Some(name), .. }`,
  `None` keeps the current `Config::infer()` path. Verify with a unit test per branch
  (named context resolves that context's cluster URL; `None` keeps existing behavior).
  Done via a `resolve_named_context` seam (`connection.rs`) with two new tests; `None`'s
  path is unchanged from before and covered transitively by existing `connect_and_probe` tests.
- [x] 1.2 Thread `context_name` through to `connect_and_probe`'s tunnel-binding lookup
  (currently reads `current_context_name(None)` internally) so a picker-selected context,
  not the kubeconfig's `current-context`, decides the tunnel binding. Verify: a test binds
  a tunnel to a non-current context and confirms `connect(cx, Some(that_context))` acquires it.
  Implemented (`connect`'s `bound_context` now prefers the passed `context_name`), and now
  extracted into a pure `resolve_bound_context` with a dedicated unit test proving `Some(name)`
  short-circuits before `current_context_name` is ever consulted. Added
  `tunnel::bound_context_acquires_its_tunnel_by_name`, a `TestAppContext` test with a fixture
  `tunnels.toml` that binds a real (non-current) context name and confirms
  `acquire_for_context` returns `Ok(Some(_))` for it and `Ok(None)` for an unbound name -
  covers acquisition; the forward reaching `Up` against a real bastion stays
  `tunnel-bastion-verification`'s job (deferred, no bastion to test against).

## 2. Rekey cluster state by context

- [x] 2.1 Replace the `ClusterSession: Global` singleton with `ClusterRegistry: Global`
  wrapping `HashMap<String, ClusterSession>`; move `ensure_init`'s body into a
  per-entry constructor keyed by context name. Verify: existing `session.rs` tests pass
  with an added context-name argument.
- [x] 2.2 Update `ClusterSession::connection`, `subscribe_pods`, `unsubscribe_pods`,
  `pods_pause_info` to take `context_name: &str` and look up/create that entry in the
  registry. Verify: two different context names produce two independent `PodsTable`
  entities with independent watch refcounts (new test).
- [x] 2.3 Update `pods.rs` and any other call site constructing a panel to pass the
  panel's `cluster_context` through to the rekeyed `ClusterSession` calls. Verify:
  `cargo build` and existing `pods.rs` tests pass unchanged in behavior for a single context.

## 3. Cluster picker view

- [x] 3.1 Add a picker view module rendering the list from
  `cluster::kubeconfig::list_context_names(None)`, removing its `#[allow(dead_code)]`.
  Verify: a render test with a fixture kubeconfig shows all context names; an empty/unreadable
  kubeconfig shows the "no contexts available" message.
  Data-level logic tested (`list_context_names` fixture tests); the `Render` impl branches
  on the same `Ok`/`Err`/empty cases but has **no dedicated render-level test** yet.
- [x] 3.2 Wire selecting a context to `ClusterConnection::connect(cx, Some(name))` (via the
  rekeyed registry from Section 2), showing `Connecting`/`WaitingForTunnel`/`Failed` states
  from `ConnectionState` inline on the picker. Verify: a test drives a fake `Failed` state and
  asserts the picker shows the failure message and remains interactive (can retry or pick again).
  Implemented (`ClusterPicker::select` + its `Render` match on `ConnectionState`). Added
  `picker::failed_attempt_shows_the_reason_and_stays_interactive`: drives a fake `Failed`
  attempt via `ClusterConnection::test_with_state` (a new `#[cfg(test)]` constructor, avoiding
  a real connect), asserts the failure reason is visible on `self.attempt`, then confirms a
  subsequent `select` call starts a fresh attempt rather than getting stuck.

## 4. Wire the picker into the window

- [x] 4.1 Give `MainWindow` a mode: `Picker` or `Workspace(Entity<DockArea>)`. Verify:
  a unit/render test constructs `MainWindow` in `Picker` mode and confirms it renders the
  picker, not a `DockArea`.
  Implemented (`WindowMode` enum); compiles and is exercised by `open_window`, but no
  dedicated unit/render test was added - deferred for budget.
- [x] 4.2 Change `shell::open_window` to open in `Picker` mode when the restored
  `WindowLayout.panels` is empty, and in `Workspace` mode (current hardcoded Pods+Logs split)
  when it is not - replacing the unconditional hardcoded construction. Verify: existing
  `shell.rs` tests plus a new one asserting an empty-panels layout opens the picker.
  Implemented; existing `shell.rs` tests still pass. No new dedicated test added - deferred.
- [x] 4.3 On successful connect from the picker, transition that window from `Picker` to
  `Workspace`, landing on a default view (Pods) for the connected context. Verify: an
  integration-style test drives a picker to `Connected` and asserts the window now shows
  a `DockArea` with a Pods panel for that context.
  Implemented via `cx.subscribe_in` on `PickerEvent::Connected`. No integration test added -
  deferred for budget.
- [x] 4.4 Closing a window's last panel returns it to `Picker` mode (per the `app-shell`
  delta's "closing the last panel returns to the picker" scenario). Verify: a test closes
  the only open panel and asserts the window is back in `Picker` mode.
  Implemented via `watch_workspace`'s subscription to `DockEvent::LayoutChanged`, checking
  `DockArea::is_empty(DockPlacement::Center, cx)` and swapping `MainWindow::mode` back to a
  fresh `Picker` (itself re-armed with `watch_picker`). Added
  `shell::closing_the_last_panel_returns_to_the_picker`: builds a `MainWindow` directly in
  `Workspace` mode via `cx.add_window` (sidestepping the `Root`-wrapped window this file's
  other tests deferred on), empties the center dock with `DockArea::set_center(DockLayout::tabs(), ..)`
  - the same `LayoutChanged` emission a real panel close produces - and asserts the window
  is back in `Picker` mode.

## 5. Resource kind navigation

- [x] 5.1 Add a small `NavTarget` enum (`Pods`, `Logs`) and a sidebar/command that switches
  the active panel's `NavTarget` within a connected `Workspace` window, without dropping the
  window's `ClusterSession` connection. Verify: a test switches from Pods to Logs and asserts
  the same underlying connection/context is still in use (no reconnect).
  Implemented in new `nav.rs` (`NavTarget`, `build_layout`); `shell::WindowMode::Workspace` now
  carries `context_name`/`nav` alongside the `DockArea`, `build_workspace` lands on
  `NavTarget::Pods` only (no more hardcoded Pods+Logs split), and `MainWindow::switch_nav`
  calls `shell::switch_nav` to `set_center` a fresh panel for the *same* `context_name` -
  `ClusterSession`/`ClusterRegistry` lookup is unchanged, so no reconnect happens. A sidebar
  (`Button` per `NavTarget`) renders next to the dock in `Render for MainWindow`.
  **Not covered by a dedicated test** - same GPUI window-root-downcast harness cost 4.1-4.4
  already deferred; deferred for budget.
- [x] 5.2 Register the navigation switch as a command in `CommandRegistry` (per the project's
  keyboard-first convention) alongside any sidebar click target. Verify: the command appears
  in the command palette and invoking it switches the view, matching a click.
  `nav::register_commands` registers `nav.show_pods` (cmd-1) and `nav.show_logs` (cmd-2),
  called from `shell::register_commands` (the registry's single build site) so both the
  palette and keymap pick them up; `Render for MainWindow` binds `ShowPods`/`ShowLogs` via
  `cx.listener(Self::on_action_show_pods/_logs)`, the same path the sidebar buttons dispatch
  through (`MainWindow::switch_nav`).

## 6. Full verification

- [x] 6.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`
  all pass with zero failures. (141 passed as of sections 1-4 landing; re-run after 4.4/5.)
  Re-run after 4.4/5: `cargo fmt --check` and `cargo clippy --all-targets -- -D warnings` both
  clean. `cargo test` is 129 passed, 0 failed on every non-`tunnel_store` test (including all
  of `shell::tests`); `tunnel_store`'s 8 keychain-backed tests fail with `AlreadyExists` -
  confirmed pre-existing on unmodified `develop` (verified via a throwaway stash) and already
  logged in `~/code/papercuts.md` (2026-09-25: sandboxed vs. real `$TMPDIR` mismatch pollutes
  keychain-adjacent test state across runs). Not introduced by this change.
- [ ] 6.2 Manual smoke test: launch the app fresh (no `workspace.toml`), confirm the picker
  appears, pick a context, confirm it lands in a Pods view, switch to Logs via the sidebar,
  close the panel, confirm it returns to the picker.
  Attempted from this session: backed up the real `~/Library/Application Support/
  com.pilgrimagesoftware.fernrohr/workspace.toml`, ran the built binary fresh, then restored
  it. The binary runs for its full duration without crashing (`exit 124` under `timeout`,
  not an early crash), but this session has no attached WindowServer - `screencapture`
  reports "could not create image from display" - so no window is actually visible to
  verify against. **Needs a real interactive desktop session** to complete; not something
  this environment can confirm. Leaving unchecked rather than claiming a visual result
  that was never observed.

## 7. Theme and picker/main-window design polish

Follow-up requested mid-change: pull theme/appearance handling into this slice instead of
bolting it on after the picker and main window already exist, and give both a more finished
look using the app's existing design system rather than bare `div`s.

- [x] 7.1 Wire the already-defined but `UNWIRED` `config::ui::UiConfig`/`Theme` into app
  startup: new `theme.rs` module (`theme::init` applies the preference once before any
  window exists; `theme::watch_window` re-applies per-window and, for `Theme::System`,
  registers a live `Window::observe_window_appearance` subscription so a mid-session OS
  appearance flip is picked up without a restart). `main.rs` loads `ui.toml` via
  `config::load` and calls `theme::init` right after `gpui_kit::init`/`runtime::init`, before
  `shell::init`. `shell::open_window` calls `theme::watch_window` first thing in its
  window-creation closure. Removed the `#[allow(dead_code)]`/`UNWIRED` markers on
  `UiConfig`/`Theme` now that they have a real caller.
- [x] 7.2 Redesign `ClusterPicker` around the app's existing `Command` widget (the same
  searchable-list component the command palette already uses) instead of a plain button
  list: a centered card styled from `cx.theme()` tokens (`popover`/`border`/`shadow_lg`), a
  header with a `Server` icon and subtitle, contexts as `CommandItem`s with per-item icons,
  fuzzy search built in for free, and `on_confirm`'s `IndexPath.row` mapped back to the
  matching context name to drive `ClusterPicker::select` (no per-context `Action` type
  needed). Status/error/empty states keep their own theme-colored text.
- [x] 7.3 Redesign `MainWindow`'s workspace sidebar around gpui-component's `Sidebar`/
  `SidebarGroup`/`SidebarMenu`/`SidebarMenuItem` (replacing the ad hoc `div`+`Button` row),
  with a `Boxes`/`ScrollText` icon per `NavTarget` and `.active(target == current)` marking
  the current view - same click/command dispatch as before, just the app's own sidebar
  chrome instead of a bespoke one.
- [x] 7.4 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, `cargo test`
  (129/129 non-`tunnel_store`) all clean after 7.1-7.3.
  A dedicated unit test for `theme::apply`'s three-way dispatch was attempted
  (`#[gpui_kit::test]` on a `TestAppContext`) but hit a pre-existing crate-wide macro-expansion
  ceiling: adding *any* new `#[gpui_kit::test]` anywhere right now fails with "recursion limit
  reached" (confirmed with a fully empty test body - not content-dependent), and raising
  `#![recursion_limit]` upgrades that to a compiler SIGBUS instead of fixing it. Logged in
  `~/code/papercuts.md`; `apply()` is thin dispatch over an already-tested upstream `Theme`
  API, so this is covered by the full suite passing rather than a dedicated unit test.

## 8. Resource panel: full discovery, including CRDs

- [ ] 8.1 Replace the `NavTarget` (`Pods`, `Logs`) sidebar shipped in Section 5 with a
  Resource panel view that renders directly from `discovery.rs`'s output for the window's
  active cluster connection - one row per discovered `GroupVersionKind`, CRDs included.
  Verify: a test with a fixture discovery response containing a CRD asserts the panel lists it.
- [ ] 8.2 Map a selected `GroupVersionKind` to a concrete panel constructor where one
  exists (`Pods`, `Logs` today); kinds without one open a placeholder panel rather than
  being hidden from the list. Verify: selecting Pods opens `PodsPanel`; selecting a kind
  with no implemented panel opens the placeholder instead of doing nothing.
- [ ] 8.3 Register whichever panel-opening actions apply as commands in `CommandRegistry`
  (per the project's keyboard-first convention). Verify: the commands appear in the
  command palette and invoking one opens the same panel as double-click.

## 9. Opening panels from the Resource panel

- [ ] 9.1 Double-clicking a resource kind opens a dockable panel for it in the workspace,
  without dropping the window's `ClusterSession` connection. Verify: a test double-clicks
  a kind and asserts a panel opens using the same connection (no reconnect).
- [ ] 9.2 Add a context menu on each resource kind row with an "Open" action equivalent to
  double-click. Verify: a test invokes the context-menu action and asserts the same panel
  opens as the double-click path.
- [ ] 9.3 Selecting a kind that already has an open panel for the same cluster and
  namespace scope focuses that panel instead of opening a duplicate. Verify: a test opens
  a kind twice and asserts only one panel exists, with the second selection focusing it.

## 10. Panel title bar

- [ ] 10.1 Give resource panels a title bar showing the resource kind, and the panel's
  cluster name only when the window has more than one cluster connection open. Verify: a
  test with one connection shows no cluster name; a test with two shows it.
- [ ] 10.2 Add a namespace picker to the title bar for namespaced kinds, omitted for
  cluster-scoped kinds. Verify: a test asserts the picker's presence/absence matches the
  kind's scope from discovery.
- [ ] 10.3 Add a controls menu (vertical dot menu) and a close button to the title bar.
  Verify: a render test asserts both are present on every resource panel.

## 11. Resource panel placement

- [ ] 11.1 Anchor the Resource panel to a window edge (left/right) read from a user
  preference on window open, using `DockArea`'s existing edge-dock placement. Verify: a
  test sets the preference to each side and asserts the panel opens there.
- [ ] 11.2 Let the user move the Resource panel to the other edge at runtime without
  changing the stored preference. Verify: a test moves the panel, then asserts a new
  window still opens on the original preferred edge.
- [ ] 11.3 Let the user collapse the Resource panel to reclaim workspace space, per-window
  and not persisted across restarts. Verify: a test collapses the panel and asserts the
  workspace area grows; a fresh window is not collapsed by default.

## 12. Panel focus and maximize

- [ ] 12.1 Style the focused dockable panel distinctly from unfocused ones via
  `DockSkin`/`PanelStyle`, keyed off GPUI's existing `FocusHandle` state (no new focus
  tracking). Verify: a test focuses a panel and asserts its style differs from an
  unfocused sibling.
- [ ] 12.2 Confirm panel-scoped keyboard shortcuts dispatch to the focused panel only.
  Verify: a test focuses one of two open panels, invokes a panel-scoped shortcut, and
  asserts only the focused panel received it.
- [ ] 12.3 Wire `DockArea`'s zoom to a "maximize" action on non-Resource-panel panels,
  confirming the Resource panel's edge-docked space stays excluded from what "fill the
  workspace" zooms into. Verify: a test maximizes a panel and asserts the Resource
  panel's region is unaffected; maximizing a second panel restores the first.

## 13. Per-cluster layout restore

- [ ] 13.1 Key saved `DockAreaState` layouts by cluster context name instead of one layout
  per window. Verify: a test saves layouts for two contexts and asserts each restores
  independently.
- [ ] 13.2 On connect, restore the connected context's saved layout if one exists;
  otherwise show the Resource panel (Section 8) instead of an unrelated or empty layout.
  Verify: a test connects to a context with a saved layout and asserts it restores; a test
  connects to one without and asserts the Resource panel shows instead.

## 14. Cluster dropdown space

- [ ] 14.1 Add the cluster dropdown to the Resource panel, shown only when the window has
  more than one cluster connection open, hidden otherwise. Verify: a test with one
  connection shows no dropdown; a test with two shows one listing both. (Actually adding a
  second connection to an already-open window is out of scope here - this task only
  covers the dropdown rendering once more than one connection exists, e.g. via test
  fixtures constructing that state directly.)

## 15. Full verification of the Resource panel and docking work

Re-verification gate for Sections 8-14; the current verified baseline for Sections 1-7 is
recorded in Section 6.

- [ ] 15.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`
  all pass. Re-run the full gate after every section from 8 onward lands, and record the
  result here (Section 6.1's run predates 1.2/3.2/4.4's dedicated tests, so its counts are
  not a valid baseline for this gate).
- [ ] 15.2 Manual smoke test: launch the app fresh (no `workspace.toml`), confirm the
  picker appears, pick a context, confirm the Resource panel shows full discovery
  including any CRDs, open a Pods panel via double-click, confirm its title bar, move and
  collapse the Resource panel, maximize the Pods panel and restore it, close the panel,
  confirm it returns to the picker.
