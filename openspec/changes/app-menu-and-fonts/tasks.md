# Tasks

## 1. Command registry menu assignment

- [x] 1.1 Added `pub menu: Option<MenuSlot>` to `Command` and `MenuSlot` (seven variants: `App`,
  `Context`, `Edit`, `View`, `Navigate`, `Window`, `Help`). Every existing `Command` construction
  site updated to `menu: None` explicitly. `CommandRegistry::for_menu(slot)` added. Verify:
  `registry_items_only_returns_the_requested_slot` in `ui::menu`'s tests.
- [x] 1.2 Assigned existing commands: `nav.show_pods`/`nav.show_logs` -> `Navigate`,
  `shell.new_window` -> `Window`, `shell.toggle_command_palette` -> `View`. Panel-scoped
  shortcuts (`w`/`d`/`l`/`y` on a pod row, the pod-detail YAML toggle) left unassigned - palette/
  keymap-only, per plan. `MenuSlot::Context` has no assigned command yet - Context menu items
  (open the cluster picker, switch context) don't exist as standalone commands in this codebase
  yet; noted as follow-up rather than inventing a command just to fill the menu.

## 2. Menu bar construction

- [x] 2.1 Built the seven-menu bar in the new `ui::menu` module, called from `shell::init` right
  before the registry moves into its global slot (menu construction needs to borrow it after
  every command is registered, before the move). Verify: build/test - GPUI's real menu bar has no
  introspectable test harness in this codebase (same class of limitation as the dock's tab
  strip), so this is verified by the manual smoke test (4.2) rather than an automated one.
- [x] 2.2 Every registry-sourced menu item constructs `MenuItem::Action` directly from the
  command's own `action.boxed_clone()` - literally the same `Box<dyn Action>` the palette
  dispatches, not a second action instance that could drift.
- [x] 2.3 Added `Quit`/`About` under App (`cx.quit()`; `About` opens a small real window showing
  `CARGO_PKG_NAME`/`CARGO_PKG_VERSION` - not a modal, since this app has no modal/dialog system
  yet and building one only for this would be its own change) and `Minimize`/`Zoom` under Window
  (dispatched against `cx.active_window()`). Verify: `quit_and_about_are_distinct_actions` in
  `ui::menu`'s tests; the window-level actions need the manual pass (4.2).

## 3. Fonts

- [x] 3.1 Sourced Manrope's variable-weight TTF from the Google Fonts mirror
  (`googlefonts/manrope` upstream, SIL OFL - both the font and `Manrope-OFL.txt` license live in
  `app/assets/fonts/`), bundled via `include_bytes!` and `cx.text_system().add_fonts(...)` in
  `theme::init` - no `AssetSource` plumbing needed, `add_fonts` takes raw bytes directly. Noted in
  design.md: the file's legacy name-table family is `Manrope ExtraLight` (its typographic/
  preferred family, name ID 16, is `Manrope`) - macOS's CoreText backend (`zed-font-kit`)
  resolves the preferred family for variable fonts, so `"Manrope"` should resolve correctly on
  the primary target platform; this is the one piece of this task that needs the manual check
  (4.2) to fully confirm, since the test harness's font system doesn't enumerate real fonts.
- [x] 3.2 `ui::theme::apply` calls `apply_fonts` after every `Theme::change`/
  `sync_system_appearance` (both the `init`-time call and every `watch_window` appearance-change
  callback), setting `font_family` to Manrope and `mono_font_family` via `first_installed_mono_font`
  (Monaco, then Menlo/Cascadia Mono/Noto Sans Mono/Liberation Mono/Ubuntu Mono/Courier New, in
  order). Verify: `every_preference_ends_with_manrope_and_a_real_mono_font` drives all three
  `ThemePreference` variants and asserts both fields.
- [x] 3.3 `first_installed_mono_font` mirrors gpui-component's own `mono_font.rs` probe pattern
  exactly (checked against installed names via `cx.text_system().all_font_names()`, never
  assigning a name that isn't present) rather than assuming Monaco - confirmed by reading that
  module's source directly, since it's the same problem gpui-component itself already solved for
  its *own* platform-default probing. Verify: the test above confirms the fallback path doesn't
  panic and produces a non-empty family in the test harness (whose font system reports no
  installed fonts, so it correctly falls through every alternate); confirming an *installed*
  Monaco is actually picked over the fallback list needs a real desktop session.

## 4. Full verification

- [x] 4.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` all
  pass (200/200, keychain-lock tests excluded per the pre-existing hang logged elsewhere).
- [ ] 4.2 Manual smoke test: launch the app, confirm the seven menus appear in order with App's
  Quit/About and Window's Minimize/Zoom present, confirm at least one menu item's action matches
  its palette behavior, confirm UI text renders in Manrope and a YAML/log view renders in Monaco
  (or a real fallback), confirm About opens a small window with the correct name/version. **Needs
  a real interactive desktop session.**
