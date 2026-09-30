# Tasks

## 1. Section grouping

- [x] 1.1 Add `DetailSection` (`Overview`, `Containers`, `Conditions`) and a `section` field on
  `PodField`; assign it at push-time in `pod_fields` per design.md's grouping. Verify: a test
  asserts each existing field's section matches the design.md table.

## 2. Tab UI

- [x] 2.1 Confirm which gpui-component tab primitive fits a plain content-switching use (not the
  dock's panel-management one) before wiring anything. Verify: a spike/read of the component's
  public API, not a task with its own test.
- [x] 2.2 Render the structured view as tabs, each showing only its section's fields via
  `render_field`. Verify: a test switches tabs and asserts only that section's field labels are
  present in the rendered output (or the closest available render-level check).
- [x] 2.3 Tab switching is keystroke-reachable, following this panel's existing hint-bar/keybind
  convention (from the YAML-toggle relocation). Verify: a test dispatches the switch-tab
  keybinding and asserts the active tab changed.

## 3. Full verification

- [x] 3.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` all
  pass.
- [ ] 3.2 Manual smoke test: open a pod's detail, confirm tabs group fields sensibly, switch tabs
  by click and by keyboard, confirm nothing from the flat list is missing.
