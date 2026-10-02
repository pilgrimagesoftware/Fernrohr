# Tasks

## 1. Per-tab close

- [ ] 1.1 Enable gpui-kit 0.7's per-tab close control on every dock tab group the app builds, and confirm it closes exactly its own tab. Verify with a window-level test that clicks the drawn close control on a background tab.
- [ ] 1.2 Make any group-level close (toolbar or menu) target its own group's active panel rather than the focused panel. Verify with the stacked-groups scenario as a window-level click test that fails before the fix.
- [ ] 1.3 Hand focus on after a mouse close the same way `Cmd-W` does. Verify the closed-unfocused-group case keeps focus where it was, and the closed-focused case moves it.

## 2. Last panel

- [ ] 2.1 Find what refuses to close the last panel (a locked or last-panel guard in the dock, or our own close handling) and allow it; the window returns to the picker. Verify with a window-level test for both the close control and `Cmd-W`.

## 3. Verification

- [ ] 3.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` pass.
- [ ] 3.2 Manual check: stacked groups, close the unfocused group's tab; close a background tab; close the last panel and land on the picker.
