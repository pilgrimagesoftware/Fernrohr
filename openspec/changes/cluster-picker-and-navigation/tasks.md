# Tasks

## 1. Connect to a named context

- [ ] 1.1 Change `ClusterConnection::connect` to `connect(cx, context_name: Option<String>)`;
  `Some(name)` builds `Config` via `Kubeconfig::read()` + `KubeConfigOptions { context: Some(name), .. }`,
  `None` keeps the current `Config::infer()` path. Verify with a unit test per branch
  (named context resolves that context's cluster URL; `None` keeps existing behavior).
- [ ] 1.2 Thread `context_name` through to `connect_and_probe`'s tunnel-binding lookup
  (currently reads `current_context_name(None)` internally) so a picker-selected context,
  not the kubeconfig's `current-context`, decides the tunnel binding. Verify: a test binds
  a tunnel to a non-current context and confirms `connect(cx, Some(that_context))` acquires it.

## 2. Rekey cluster state by context

- [ ] 2.1 Replace the `ClusterSession: Global` singleton with `ClusterRegistry: Global`
  wrapping `HashMap<String, ClusterSession>`; move `ensure_init`'s body into a
  per-entry constructor keyed by context name. Verify: existing `session.rs` tests pass
  with an added context-name argument.
- [ ] 2.2 Update `ClusterSession::connection`, `subscribe_pods`, `unsubscribe_pods`,
  `pods_pause_info` to take `context_name: &str` and look up/create that entry in the
  registry. Verify: two different context names produce two independent `PodsTable`
  entities with independent watch refcounts (new test).
- [ ] 2.3 Update `pods.rs` and any other call site constructing a panel to pass the
  panel's `cluster_context` through to the rekeyed `ClusterSession` calls. Verify:
  `cargo build` and existing `pods.rs` tests pass unchanged in behavior for a single context.

## 3. Cluster picker view

- [ ] 3.1 Add a picker view module rendering the list from
  `cluster::kubeconfig::list_context_names(None)`, removing its `#[allow(dead_code)]`.
  Verify: a render test with a fixture kubeconfig shows all context names; an empty/unreadable
  kubeconfig shows the "no contexts available" message.
- [ ] 3.2 Wire selecting a context to `ClusterConnection::connect(cx, Some(name))` (via the
  rekeyed registry from Section 2), showing `Connecting`/`WaitingForTunnel`/`Failed` states
  from `ConnectionState` inline on the picker. Verify: a test drives a fake `Failed` state and
  asserts the picker shows the failure message and remains interactive (can retry or pick again).

## 4. Wire the picker into the window

- [ ] 4.1 Give `MainWindow` a mode: `Picker` or `Workspace(Entity<DockArea>)`. Verify:
  a unit/render test constructs `MainWindow` in `Picker` mode and confirms it renders the
  picker, not a `DockArea`.
- [ ] 4.2 Change `shell::open_window` to open in `Picker` mode when the restored
  `WindowLayout.panels` is empty, and in `Workspace` mode (current hardcoded Pods+Logs split)
  when it is not - replacing the unconditional hardcoded construction. Verify: existing
  `shell.rs` tests plus a new one asserting an empty-panels layout opens the picker.
- [ ] 4.3 On successful connect from the picker, transition that window from `Picker` to
  `Workspace`, landing on a default view (Pods) for the connected context. Verify: an
  integration-style test drives a picker to `Connected` and asserts the window now shows
  a `DockArea` with a Pods panel for that context.
- [ ] 4.4 Closing a window's last panel returns it to `Picker` mode (per the `app-shell`
  delta's "closing the last panel returns to the picker" scenario). Verify: a test closes
  the only open panel and asserts the window is back in `Picker` mode.

## 5. Resource kind navigation

- [ ] 5.1 Add a small `NavTarget` enum (`Pods`, `Logs`) and a sidebar/command that switches
  the active panel's `NavTarget` within a connected `Workspace` window, without dropping the
  window's `ClusterSession` connection. Verify: a test switches from Pods to Logs and asserts
  the same underlying connection/context is still in use (no reconnect).
- [ ] 5.2 Register the navigation switch as a command in `CommandRegistry` (per the project's
  keyboard-first convention) alongside any sidebar click target. Verify: the command appears
  in the command palette and invoking it switches the view, matching a click.

## 6. Full verification

- [ ] 6.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`
  all pass with zero failures.
- [ ] 6.2 Manual smoke test: launch the app fresh (no `workspace.toml`), confirm the picker
  appears, pick a context, confirm it lands in a Pods view, switch to Logs via the sidebar,
  close the panel, confirm it returns to the picker.
