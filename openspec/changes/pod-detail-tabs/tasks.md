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

## 3. Revised grouping

- [x] 3.1 Regroup into Overview, Containers, Volumes, Events, Managed Fields per design.md's
  revised table, each tab with its own keybinding (1-5). Verify: the section-grouping test
  matches the revised table, and the keybinding test dispatches every tab's action.
- [x] 3.2 Events tab: fetch the pod's events alongside it, scoped by kind and UID, reading
  `events.k8s.io/v1` timestamps and counts, and reporting a list failure in the tab. Verify:
  tests cover ordering across legacy and series timestamps, the selector, and a forbidden events
  list that still loads the pod.
- [x] 3.3 Managed Fields tab: one expandable block per manager with its pretty-printed `fieldsV1`.
  Verify: tests cover the projection, including an entry with no manager, operation, or fields.

## 4. Full verification

- [x] 4.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` all
  pass.
- [ ] 4.2 Manual smoke test: open a pod's detail, confirm tabs group fields sensibly, switch tabs
  by click and by keyboard (1-5), confirm nothing from the flat list is missing, and that the
  Events tab shows the pod's recent events.
