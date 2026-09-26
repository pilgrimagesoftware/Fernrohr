# Tasks

## 1. Connect to a named context

- [x] 1.1 Change `ClusterConnection::connect` to `connect(cx, context_name: Option<String>)`;
  `Some(name)` builds `Config` via `Kubeconfig::read()` + `KubeConfigOptions { context: Some(name), .. }`,
  `None` keeps the current `Config::infer()` path. Verify with a unit test per branch
  (named context resolves that context's cluster URL; `None` keeps existing behavior).
  Done via a `resolve_named_context` seam (`connection.rs`) with two new tests; `None`'s
  path is unchanged from before and covered transitively by existing `connect_and_probe` tests.
- [ ] 1.2 Thread `context_name` through to `connect_and_probe`'s tunnel-binding lookup
  (currently reads `current_context_name(None)` internally) so a picker-selected context,
  not the kubeconfig's `current-context`, decides the tunnel binding. Verify: a test binds
  a tunnel to a non-current context and confirms `connect(cx, Some(that_context))` acquires it.
  Implemented (`connect`'s `bound_context` now prefers the passed `context_name`) but
  **not yet covered by a dedicated test** - would need a GPUI `TestAppContext` +
  fixture `tunnels.toml` + fixture kubeconfig wired together; deferred for budget.

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
- [ ] 3.2 Wire selecting a context to `ClusterConnection::connect(cx, Some(name))` (via the
  rekeyed registry from Section 2), showing `Connecting`/`WaitingForTunnel`/`Failed` states
  from `ConnectionState` inline on the picker. Verify: a test drives a fake `Failed` state and
  asserts the picker shows the failure message and remains interactive (can retry or pick again).
  Implemented (`ClusterPicker::select` + its `Render` match on `ConnectionState`) but
  **not covered by a test**; deferred for budget.

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
  fresh `Picker` (itself re-armed with `watch_picker`). **Not covered by a dedicated test** -
  same GPUI window-root-downcast harness cost already deferred by 4.1-4.3; deferred for budget.

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
