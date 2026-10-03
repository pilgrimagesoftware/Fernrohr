# Tasks

## 1. Quick look popover

- [x] 1.1 Add the popover anchored to the selected Pods row, built from the shared Pod store with pod detail's projection helpers and status colors: name/namespace, phase, ready, restarts, age, node, IP, owner link, containers' images and states. Verify with a window-level test that Space on a selected crashing pod draws those fields.
- [x] 1.2 Bind Space/Escape to close and Enter/Open Details button to open the detail panel; register "Pods: Quick Look" (panel-scoped, palette, hint row). Verify with keystroke tests for open, close and Enter, and the button click.
- [x] 1.3 Up/Down move the table selection while open and the popover follows; the popover updates live and reports a deleted pod. Verify with tests for scanning, a ready-count change, and deletion.
- [x] 1.4 Latest Warning event via the per-pod event watch, started on open, retargeted (debounced) on selection change, dropped on close. Verify with a mock-server test that the warning appears and that closing releases the watch.

## 2. Row context menu

- [x] 2.1 Right-click a pod row selects it and shows Quick Look, Open Details, Logs, YAML from the registered commands. Verify with a right-click test choosing Logs.

## 3. Verification

- [x] 3.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` pass.
- [x] 3.2 Manual check: Space on a pod, scan with Down, Enter to the detail panel, right-click a row.
  Passed 2026-10-03 (user).

## Notes

- Implemented in App#116. Escape normally reaches the table's `Cancel` (which clears the selection), so while a quick look is open the Pods panel captures it to close the popover instead. A fix in `ui::list_keys::step` was needed for Up/Down to reach a row's context menu.
- 3.2 (manual check) is still open.
