# Tasks

## 1. Inset the focus border (superseded)

- [x] 1.1 Change `focus_frame` to draw its border against an inset inner element rather than the
  full-size outer one, so the border never sits flush with a panel's outer edge.
- [x] 1.2 ~~Manual check: the inset border clears the window's rounded corner.~~ Moot: the border
  itself was removed in App `198dcaa` after Paul's review, before this was checked. Section 2
  replaces it.

## 2. Focus indicator on the panel's tab

- [x] 2.1 `title_element` takes the panel's focus state and draws a 2px underline in `theme.blue`
  while focused, transparent otherwise (`focus_underline`). Unit test:
  `only_the_focused_panel_underlines_its_title`.
- [x] 2.2 Every panel's `title()` passes `focus_handle.contains_focused(window, cx)` (Pods, Pod
  detail, Logs, Placeholder).
- [x] 2.3 Placeholder and Logs track their focus handle, so a click focuses them. Tests:
  `a_click_focuses_the_placeholder`, `a_click_focuses_the_logs_panel` (each fails without
  `track_focus`).
- [x] 2.4 Dock-level test through the app's real `DockSkin` dock: of two tabbed panels, only the
  focused one's tab is underlined, and clicking the other tab moves the underline there
  (`the_focused_panels_tab_is_the_one_underlined`; fails if a panel's title ignores focus).
- [x] 2.5 Remove the no-op `focus_frame` and its four call sites.
- [x] 2.6 Move `title.rs`'s colocated tests to `title/tests.rs` to stay under the 500-line limit.
- [x] 2.7 Use the user's accent colour on macOS (`ui::accent`: `NSColor.controlAccentColor`, cached,
  re-read on startup, appearance change and window activation), falling back to `theme.blue`.
  Tests: `a_system_accent_wins_over_the_theme`, `without_a_system_accent_the_theme_blue_is_used`,
  `macos_reads_an_opaque_system_accent`.

## 3. Full verification

- [x] 3.1 `cargo fmt -- --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test`
  all pass (374 passed, 2 ignored real-keychain tests).
- [x] 3.2 Manual check against a running build: with two or more panels open, click between them
  and confirm the underline reads clearly in both light and dark mode, that no tab label shifts
  when focus moves, and that changing the accent colour in System Settings shows on switching
  back. Confirmed by Paul, 2026-09-30.

## 4. Follow-up (out of scope here)

- [x] 4.1 (Taken up by `panel-focus-navigation`.) A keyboard route for moving focus between panels (a registered command, e.g. next/previous
  panel), and deciding whether opening a panel from the keyboard should focus it. Today neither
  exists, so only the mouse can move focus from one panel to another. The indicator itself already
  follows focus from any source.
- [x] 4.2 Linux and Windows system accent lookups (today both use the theme's blue).
  Moved to follow-up issue #100 (2026-10-02).
