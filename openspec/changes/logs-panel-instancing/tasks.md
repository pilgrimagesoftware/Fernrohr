# Tasks

## 1. Per-pod logs target

- [ ] 1.1 Add `NavTarget::PodLogs(PodRef)`. Verify: a test constructs two variants for different
  pods and asserts they're unequal, the same pod twice equal - mirrors the existing
  `NavTarget::Pod` coverage.
- [ ] 1.2 `LogsPanel::new` takes `Option<PodRef>`: `Some` pins the panel to that pod with no
  `SelectedPod` observer; `None` keeps today's reactive construction. Verify: a test constructs
  a pinned panel, changes the `SelectedPod` global to a different pod, and asserts the panel's
  stream is unaffected; a second test does the same for a `None`-constructed panel and asserts it
  *does* retarget.

## 2. Preference and dispatch

- [ ] 2.1 Add `UiConfig.logs_panel_reuse: bool` (default `true`). Verify: round-trip test
  matching `UiConfig`'s existing TOML tests.
- [ ] 2.2 Add `ShowPodLogsNewPanel` action, bound to `shift-l` in `PodsPanel`'s key context
  alongside the existing `l` -> `ShowPodLogs`. Verify: a keystroke-level test (matching the
  `panel_bindings` regression-test pattern already established) asserts `shift-l` dispatches the
  new action and `l` still dispatches `ShowPodLogs`.
- [ ] 2.3 `ShowPodLogs`'s handler resolves `NavTarget::Logs` or `NavTarget::PodLogs(pod_ref)` per
  `UiConfig.logs_panel_reuse`; `ShowPodLogsNewPanel`'s handler always resolves to
  `NavTarget::PodLogs(pod_ref)`. Verify: a test with the preference set each way asserts
  `ShowPodLogs` opens the matching target; a separate test asserts `ShowPodLogsNewPanel` always
  opens `PodLogs` regardless of the preference.

## 3. Full verification

- [ ] 3.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` all
  pass.
- [ ] 3.2 Manual smoke test: with the default preference, open two pods' logs in turn and confirm
  the second replaces the first; use `shift-l` on a third pod and confirm it opens alongside
  rather than replacing; flip the preference and confirm plain `l` now opens new instances by
  default.
