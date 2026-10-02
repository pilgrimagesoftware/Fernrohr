# Tasks

## 1. Status bar capsules

- [x] 1.1 Merge the context bar's chips into the status bar's per-context items as capsules (name, tunnel, health colour and icon, state text, elapsed time), keeping problem-first ordering. Verify with window-level tests for the two-context and paused-first scenarios and that each context's name is drawn once in the chrome.
  App#98: `StatusBarView` capsules (`ui/status_bar/capsule.rs`). Verified by
  `util::shell::contexts::capsule_tests::the_capsules_are_in_the_status_bar_at_the_bottom`, the ported
  `ui::status_bar::capsule::tests` and `ui::status_bar::tests::non_connected_items_sort_before_connected_ones`.
- [x] 1.2 Move the add-context control to follow the capsules, anchoring its popover in the status bar, and keep its keyboard command. Verify the add scenarios still pass through the moved control.
  The add control follows the capsules. There was no keyboard command, so one was added: `context.add`
  (Context menu, palette). Verified by `capsule_tests::the_add_control_and_command_open_the_add_popover`.
- [x] 1.3 Move the disconnect action onto each capsule, keeping the confirmation. Verify the disconnect scenarios, including the last context returning to the picker.
  On each capsule's menu, plus the new `context.disconnect` for the active context. Verified by
  `capsule_tests::disconnect_confirms_then_removes_the_capsule_and_the_last_returns_to_the_picker` and
  `capsule_tests::a_capsules_menu_offers_disconnect`.
- [x] 1.4 Remove the context bar. Verify no context bar element is drawn.
  `ui/context_bar.rs` removed; verified by `util::shell::render::toolbar_tests` (no context in the top bar).

## 2. Theme switcher

- [x] 2.1 Add a System/Light/Dark switcher at the status bar's far end and a palette command, applying to every window and persisting the theme preference in `ui.toml`. Verify with tests that switching redraws all windows and the preference round-trips.
  `ui::theme::set`, `theme.system`/`theme.light`/`theme.dark` and the status bar switcher. Verified by
  `util::shell::app::theme_tests` (both windows switch, `ui.toml` saved with its other settings kept, a
  relaunch starts dark; the switcher's menu). Following the OS has no automated test: the test platform
  can't change the OS appearance.

## 3. Toolbar

- [x] 3.1 Replace the top bar with a gpui-kit Toolbar showing the app icon and name only, keeping the window draggable and the macOS traffic lights usable. Verify with a test that the toolbar draws icon and name and no context chips.
  `ui::toolbar::window_toolbar`: a gpui-kit `TitleBar` (drag, zoom, traffic lights) holding a `Toolbar`.
  Verified by `util::shell::render::toolbar_tests::the_toolbar_shows_the_icon_and_name_and_no_context_chips`.

## 4. Verification

- [x] 4.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` pass.
  809 passed, 2 ignored.
- [ ] 4.2 Manual check: capsules at the bottom with add and disconnect working; theme switcher changes theme live and survives restart; toolbar shows icon and name; window still drags by its top bar.
