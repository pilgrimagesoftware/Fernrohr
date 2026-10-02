# Tasks

## 1. Per-tab close

- [x] 1.1 Enable gpui-kit 0.7's per-tab close control on every dock tab group the app builds, and confirm it closes exactly its own tab. Verify with a window-level test that clicks the drawn close control on a background tab.
- [x] 1.2 Make any group-level close (toolbar or menu) target its own group's active panel rather than the focused panel. Verify with the stacked-groups scenario as a window-level click test that fails before the fix.
- [x] 1.3 Hand focus on after a mouse close the same way `Cmd-W` does. Verify the closed-unfocused-group case keeps focus where it was, and the closed-focused case moves it.

## 2. Last panel

- [x] 2.1 Find what refuses to close the last panel (a locked or last-panel guard in the dock, or our own close handling) and allow it; the window returns to the picker. (Superseded by 4.1: the window stays connected.) Verify with a window-level test for both the close control and `Cmd-W`.

## 4. Second smoke test

- [x] 4.1 Closing the last panel keeps the window connected (no picker): the Resource panel shows beside an empty area naming the key to open a kind, and takes focus, expanding if collapsed. Update `cluster-picker-and-navigation`'s spec deltas to match (done in this change's proposal commit). Verify with window-level tests for the close control and `Cmd-W`, and that disconnecting the last context still returns to the picker.
- [x] 4.2 A lone panel's close control sits beside its title, not at the group's far edge. Verify with a test on the drawn control's position relative to the title.
- [ ] 4.3 Fix focus after closing the focused panel with the Resource panel collapsed and one tab group: focus intermittently lands nowhere, and `Cmd-]`/`Cmd-0` then can't recover it. Every close must leave a focused panel (`app-shell`: the active panel tab holds keyboard focus), and focus-cycling commands must work from an unfocused window. Verify with a test that closes the focused tab repeatedly (mouse and `Cmd-W`, alternating) and asserts a panel has focus after each.

## 3. Verification

- [x] 3.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` pass.
- [ ] 3.2 Manual check: stacked groups, close the unfocused group's tab; close a background tab; close the last panel and land on the picker.
