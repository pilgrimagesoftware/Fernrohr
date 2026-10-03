# Tasks

## 1. Command and keybinding

- [ ] 1.1 Add a `ViewLogs` action to `App/app/src/k8s/resource/pod_detail/commands.rs`'s
  `actions!` list, a `VIEW_LOGS_KEY: &str = "l"` constant, and register it as
  `pod_detail.view_logs` / "Pod Detail: View Logs" in `register_commands`, scoped to
  `PANEL_KEY_CONTEXT`. Verify with `cargo build` and that the command appears in the palette
  while a pod detail panel has focus.

## 2. Action handler

- [ ] 2.1 In `App/app/src/k8s/resource/pod_detail/panel.rs` (or a sibling action-handler file,
  matching how `ToggleDetailView`/tab-select actions are already handled), add an
  `on_action_view_logs` handler: while `self.state` is `PodDetailState::Loaded(pod)`, build a
  `PodSelection { namespace: self.pod.namespace, name: self.pod.name, containers:
  pod.spec.containers names, context_name: self.scope.context_name }`, set it on the
  `SelectedPod` global, and dispatch `ShowLogs` (`crate::ui::nav::ShowLogs`). No-op while the
  panel is `Loading` or in an error state.
- [ ] 2.2 Wire the handler into the panel's action dispatch (wherever `ToggleDetailView` and the
  tab-select actions are already bound to `cx.listener` / `on_action`). Verify with a test that
  simulates the `l` keystroke on a loaded pod detail panel and asserts the Logs panel opens
  scoped to that pod (see existing keystroke-driven tests in this module, e.g. for
  `ToggleDetailView`).

## 3. Key hint and tests

- [ ] 3.1 Add the `l` hint to the pod detail panel's header hint row in `render.rs`, alongside
  the existing tab-key hints, so the shortcut is visible the same way the others are.
- [ ] 3.2 Add a unit/integration test covering a multi-container pod: invoking the command opens
  Logs defaulting to the first container, matching the existing Pods-list-to-Logs behavior.
  Verify with `cargo test`.
- [ ] 3.3 Run `cargo fmt && cargo clippy -- -D warnings && cargo test` and confirm all pass.
