# Tasks

## 1. Command registry menu assignment

- [ ] 1.1 Add `pub menu: Option<MenuSlot>` to `Command`, and a `MenuSlot` enum with the seven
  top-level menu variants (`App`, `Context`, `Edit`, `View`, `Navigate`, `Window`, `Help`).
  Default existing `Command::new`/registration call sites to `menu: None` so nothing appears in
  a menu until deliberately assigned. Verify: a test registers one command with a menu slot and
  one without, and asserts `CommandRegistry::for_menu(MenuSlot::View)` returns only the former.
- [ ] 1.2 Assign existing commands their menu slots where a sensible one exists (e.g.
  `nav.show_pods`/`nav.show_logs` under Navigate, `toggle_command_palette` under Edit or View -
  pick per GPUI/macOS convention). Leave panel-scoped shortcuts (describe/logs/yaml on a pod row)
  unassigned - they stay palette/keymap-only. Verify: existing command-registry tests still pass
  unchanged.

## 2. Menu bar construction

- [ ] 2.1 Build the seven-menu native menu bar at startup (alongside `theme::init`/`shell::init`
  in `main.rs`), with items for each menu populated from `CommandRegistry::for_menu`. Verify: a
  test (or the closest GPUI menu-inspection harness available) asserts the menu bar's top-level
  titles and order.
- [ ] 2.2 Wire each generated menu item to dispatch the same action its command palette entry
  would dispatch. Verify: a test invokes a menu item's action and asserts the same state change
  a palette invocation of that command produces.
- [ ] 2.3 Add the platform-standard non-command items that have no `CommandRegistry` entry:
  `Quit`/`About` under App, `Minimize`/`Zoom` (or GPUI's native equivalents) under Window.

## 3. Fonts

- [ ] 3.1 Source a redistributable Manrope font file (SIL OFL) and add it to the app's asset
  bundle (`gpui_kit::assets::Assets`), registered via the text system before first paint.
- [ ] 3.2 In `crate::ui::theme::apply`, after the existing `Theme::change`/
  `sync_system_appearance` call, set `font_family` to Manrope and `mono_font_family` to Monaco (with
  a real fallback family, not merely GPUI's untouched default) on every appearance change, so a
  light/dark flip cannot revert either field to the library default.
  Verify: a test drives `apply` through each `ThemePreference` variant and asserts both font
  fields hold the expected values afterward.
- [ ] 3.3 Confirm on a non-macOS (or Monaco-less) environment that the mono fallback renders a
  real installed monospace font rather than silently keeping GPUI's proportional default. Verify:
  a test or manual check with `mono_font_family` forced to a name not present on the host still
  shows monospaced text.

## 4. Full verification

- [ ] 4.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` all
  pass.
- [ ] 4.2 Manual smoke test: launch the app, confirm the seven menus appear in order, confirm at
  least one menu item's action matches its palette behavior, confirm UI text and a YAML/log view
  render in the new fonts.
