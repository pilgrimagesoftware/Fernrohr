# Tasks

## 1. Status bar capsules

- [ ] 1.1 Merge the context bar's chips into the status bar's per-context items as capsules (name, tunnel, health colour and icon, state text, elapsed time), keeping problem-first ordering. Verify with window-level tests for the two-context and paused-first scenarios and that each context's name is drawn once in the chrome.
- [ ] 1.2 Move the add-context control to follow the capsules, anchoring its popover in the status bar, and keep its keyboard command. Verify the add scenarios still pass through the moved control.
- [ ] 1.3 Move the disconnect action onto each capsule, keeping the confirmation. Verify the disconnect scenarios, including the last context returning to the picker.
- [ ] 1.4 Remove the context bar. Verify no context bar element is drawn.

## 2. Theme switcher

- [ ] 2.1 Add a System/Light/Dark switcher at the status bar's far end and a palette command, applying to every window and persisting the theme preference in `ui.toml`. Verify with tests that switching redraws all windows and the preference round-trips.

## 3. Toolbar

- [ ] 3.1 Replace the top bar with a gpui-kit Toolbar showing the app icon and name only, keeping the window draggable and the macOS traffic lights usable. Verify with a test that the toolbar draws icon and name and no context chips.

## 4. Verification

- [ ] 4.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` pass.
- [ ] 4.2 Manual check: capsules at the bottom with add and disconnect working; theme switcher changes theme live and survives restart; toolbar shows icon and name; window still drags by its top bar.
