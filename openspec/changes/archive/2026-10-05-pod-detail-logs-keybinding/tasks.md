# Tasks

## 1. Command and keybinding

- [x] 1.1 Add a `ViewLogs` action to `App/app/src/k8s/resource/pod_detail/commands.rs`'s
  `actions!` list, a `VIEW_LOGS_KEY: &str = "l"` constant, and register it as
  `pod_detail.view_logs` / "Pod Detail: View Logs" in `register_commands`, scoped to
  `PANEL_KEY_CONTEXT`. Verify with `cargo build` and that the command appears in the palette
  while a pod detail panel has focus.

## 2. Action handler

- [x] 2.1 In `App/app/src/k8s/resource/pod_detail/panel.rs` (or a sibling action-handler file,
  matching how `ToggleDetailView`/tab-select actions are already handled), add an
  `on_action_view_logs` handler: while `self.state` is `PodDetailState::Loaded(pod)`, build a
  `PodSelection { namespace: self.pod.namespace, name: self.pod.name, containers:
  pod.spec.containers names, context_name: self.scope.context_name }`, set it on the
  `SelectedPod` global, and dispatch `ShowLogs` (`crate::ui::nav::ShowLogs`). No-op while the
  panel is `Loading` or in an error state.
- [x] 2.2 Wire the handler into the panel's action dispatch (wherever `ToggleDetailView` and the
  tab-select actions are already bound to `cx.listener` / `on_action`). Verify with a test that
  simulates the `l` keystroke on a loaded pod detail panel and asserts the Logs panel opens
  scoped to that pod (see existing keystroke-driven tests in this module, e.g. for
  `ToggleDetailView`).

## 3. Key hint and tests

- [x] 3.1 Add the `l` hint to the pod detail panel's header hint row in `render.rs`, alongside
  the existing tab-key hints, so the shortcut is visible the same way the others are.
- [x] 3.2 Add a unit/integration test covering a multi-container pod: invoking the command opens
  Logs defaulting to the first container, matching the existing Pods-list-to-Logs behavior.
  Verify with `cargo test`.
- [x] 3.3 Run `cargo fmt && cargo clippy -- -D warnings && cargo test` and confirm all pass.

## Notes

- Implemented in pilgrimagesoftware/Fernrohr-App (branch `125-pod-detail-logs-keybinding`), one
  commit per section above.
- View Logs is `pod_detail.view_logs` ("Pod Detail: View Logs"), default `l`, bound in
  `PodDetailPanel && !Input` rather than the panel's plain context: a bare letter must not fire
  while a text field has focus. A keystroke test types `l` inside an `Input` context within the
  panel and checks nothing fires.
- The handler lives in `pod_detail/logs.rs` (keeping `panel.rs` under the line cap) and builds
  the same `PodSelection` the Pods list does, containers in `spec.containers` order, then
  dispatches `ShowLogs`. Before the pod loads it does nothing.
- 3.2 is covered twice: at the panel (a two-container pod publishes `[app, sidecar]`) and through
  a real window (the Logs panel opens on the pod's context with `[web, proxy]`; pressing `l` again
  focuses that panel instead of opening a second).

## Manual checks

- [x] Open a pod's detail, wait for it to load, press `l`: the Logs panel opens (or focuses) on that
      pod and streams its first container
- [x] For a pod with several containers, the Logs panel's container picker offers the others
- [x] The header's hint row shows `l Logs`, and the palette lists "Pod Detail: View Logs" while the
      detail panel has focus
