# Tasks

## 1. Saved-layout file store

- [ ] 1.1 Add `SavedLayout { version, name, created_at, updated_at, contexts, dock: DockAreaState,
  resource_panel_width: Option<f32>, window_width, window_height }` in a new
  `app/src/config/saved_layouts.rs`, mirroring `dock_layouts.rs`'s placement (per design.md D2/D3).
  Verify with a unit test: a `SavedLayout` round-trips through `serde_json`.
- [ ] 1.2 Add the filename-derivation helper (slugify a display name; fixed placeholder stem for a
  name that slugifies to nothing) per design.md D2. Verify with unit tests: `"My Layout"` and
  `"my layout"` derive the same stem; a name of only punctuation derives the placeholder stem.
- [ ] 1.3 Add `save(dir: &Path, layout: &SavedLayout) -> io::Result<PathBuf>`: derives the filename,
  disambiguates a collision with a *different* saved name by appending `-2`, `-3`, ... (per design.md
  D2), creates `dir` with `fs::create_dir_all` if missing, and writes atomically (temp file in `dir`,
  then `fs::rename` onto the final path). Verify with unit tests: saving creates the directory on
  first use; two different names that slugify alike get distinct files; an interrupted write (temp
  file left behind, final file never written) is simulated and asserted not to have touched any
  existing file of the same derived name.
- [ ] 1.4 Add `load_all(dir: &Path) -> (Vec<SavedLayout>, Vec<UnreadableLayout>)`, where
  `UnreadableLayout` carries the filename of a file that failed to read or parse. Verify with a unit
  test: a directory with two valid files and one corrupt file returns both valid layouts and one
  `UnreadableLayout` naming the corrupt file, and the corrupt file's bytes are unchanged on disk
  afterward.
- [ ] 1.5 Add `rename(dir, old_name, new_name) -> Result<(), NameTaken>` (case-insensitive name
  collision check per design.md D2) and `remove(dir, name) -> io::Result<()>`. A rename whose derived
  filename changes writes the new file before removing the old one. Verify with unit tests: renaming
  to a name already in use (including a same-only-by-case match) returns `NameTaken` and leaves both
  files unchanged; renaming to a free name updates the file's `name` field and, when the derived
  filename changed, leaves exactly one file (the new one) in the directory; removing an absent name is
  a no-op.

## 2. Save Panel Layout command

- [ ] 2.1 Register the `layouts.save` command (id, title "Save Panel Layout…", default binding
  `cmd-shift-s`, context `Workspace`, menu `MenuSlot::Window`) wired to a naming dialog (text input,
  Tab-reachable, Enter confirms, Escape cancels). Verify: a `VisualTestContext::simulate_keystrokes`
  test opens the dialog via the default binding and confirms it appears; a render test confirms it
  does not appear while a window is in cluster-picker mode (no `Workspace` context active).
- [ ] 2.2 Wire the dialog's confirm to capture the current window's dock (`dock_area.read(cx).dump
  (cx)`), Resource panel width, contexts, and window bounds (`restorable_bounds`/`layout_from_window`'s
  existing helpers) into a `SavedLayout`, calling `config::saved_layouts::save`. Verify: a test saves
  a window with two panels and asserts the resulting `SavedLayout`'s dock JSON contains both panels'
  content keys and the window's current bounds.
- [ ] 2.3 Add the overwrite-confirmation step from the spec's "Saving over an existing name" scenario
  (case-insensitive name match against the saved layouts in `layouts/`). Verify: a keystroke test
  enters a name matching an existing saved layout's name only by case, asserts a confirmation prompt
  appears, and that declining it leaves the existing saved layout's `updated_at` unchanged.
- [ ] 2.4 Add the structural secret-safety regression test from design.md D8: reveal a `SecretValue`
  in a panel, save a layout, and assert `format!("{:?}", saved_layout)` (and the serialized JSON)
  contains no fixture secret value.

## 3. Saved Layouts picker (list, rename, delete)

- [ ] 3.1 Add a `SavedLayoutsPicker` view under `app/src/ui/picker/`, following `ClusterPicker`'s
  structure (state/interaction/layout/rows/render split, `window.last_input_was_keyboard()` to
  distinguish hover from keyboard selection), listing every `SavedLayout` by name via
  `config::saved_layouts::load_all`. Verify: a render test with a fixture directory of two saved
  layouts shows both names; an empty directory shows an explicit "no saved layouts" message rather
  than a blank list; a directory with one unreadable file shows the readable layouts plus a notice
  naming the unreadable file.
- [ ] 3.2 Register `layouts.manage` (title "Saved Layouts…", default binding `cmd-shift-o`, context
  `None`, menu `MenuSlot::Window`) opening the picker, available from both `Workspace` and
  cluster-picker window modes. Verify: a keystroke test opens the picker via the default binding
  from a window in cluster-picker mode.
- [ ] 3.3 Confirm hover never moves the keyboard-selected row: wire row hover through
  `last_input_was_keyboard()` the same way `ClusterPicker` does, not through the underlying
  `Command` widget's own hover-selects behavior. Verify: a test simulates arrow-key navigation to
  select row 2, then simulates a mouse-hover event over row 1, and asserts row 2 is still selected.
- [ ] 3.4 Register `saved_layouts.rename_selected` (default binding `r`, context
  `SavedLayoutsPicker`) opening an inline rename field; reject a colliding rename via
  `config::saved_layouts::rename`'s `NameTaken`. Verify: a keystroke test renames the selected row by
  pressing `r`, typing a new name, and Enter, then asserts the picker lists the new name; a second
  test attempts a colliding rename (including one differing only by case) and asserts both original
  names remain.
- [ ] 3.5 Register `saved_layouts.delete_selected` (default binding `backspace`, context
  `SavedLayoutsPicker`), calling `confirm_dialog::open` with `Severity::Irreversible` (design.md D6),
  then `config::saved_layouts::remove` on confirm. Verify: a keystroke test presses backspace,
  asserts the confirmation opens with Cancel focused, presses Enter, and asserts the layout is NOT
  removed (Enter cancels an Irreversible confirmation); a second test presses backspace, then
  activates the confirm button (click, or Tab to it and press Enter/Space, or the
  `dialog.confirm_irreversible` shortcut), and asserts the row is gone from the picker and the file is
  gone from `layouts/`.

## 4. Loading a saved layout: Add and Replace

- [ ] 4.1 Register `saved_layouts.load_replace` (default binding `enter`, context
  `SavedLayoutsPicker`): closes the window's current panels and rebuilds its dock via `DockArea::load`
  from the saved `DockAreaState`, then applies the saved Resource panel width and window bounds
  (`window_bounds`'s existing clamping/centering fallback). Verify: a test opens the picker from a
  window with one arrangement, loads a different saved layout with Replace, and asserts the window's
  dock, Resource panel width, and bounds now match the saved layout, not the prior state.
- [ ] 4.2 Register `saved_layouts.load_add` (default binding `secondary-enter`, context
  `SavedLayoutsPicker`): decodes the saved layout's panels via `restored_panel_keys`/`panel_key` (per
  design.md D3) and opens each one through `MainWindow::open_target_in`, leaving the window's existing
  panels, Resource panel state, and bounds untouched. Verify: a test loads with Add into a window that
  already has one panel open and asserts the window now shows both the original panel and the saved
  layout's panels, with Resource panel width and window bounds unchanged; a second test loads with Add
  a saved layout containing a panel whose content key matches a panel already open and asserts no
  duplicate panel is created.
- [ ] 4.3 Because `app/src/util/shell/panels.rs`'s `PanelKey`/`restored_panel_keys` are `pub(super)`,
  add the capture/apply logic (2.2's dock capture, 4.1's Replace, 4.2's Add) as a new sibling module
  `app/src/util/shell/saved_layouts.rs` inside `util::shell` (per design.md D3), not as a widened-
  visibility export. Verify: `cargo build` succeeds with no visibility widened beyond `pub(super)`/
  `pub(in crate::util::shell)` for these items; a shared-fixture test asserts the same `PanelKey`
  values decode whether the source is `dock-layouts.json` or a `SavedLayout.dock`.

## 5. Missing context, namespace, object, and panel-kind handling on restore

- [ ] 5.1 For each panel decoded while loading (Add or Replace), check its `context_name` against
  the window's currently held contexts (`WindowMode::Workspace`'s `contexts`); a context not held
  restores that one panel as a placeholder naming the context and stating this window isn't
  connected to it, without opening it via `open_target_in` and without adding the context to the
  window (per design.md D5/Non-Goals). Verify: a test loads a layout referencing a context the window
  does not hold and asserts that panel renders as a placeholder, while every other panel in the
  layout restores normally; a test confirms the window's held-context list is unchanged after the
  load.
- [ ] 5.2 Confirm a restored panel whose namespace or object no longer exists uses each panel kind's
  own existing not-found handling (e.g. `pod-detail`'s "pod no longer exists" requirement), with no
  new per-saved-layout logic required. Verify: a test loads a layout whose pod detail panel points at
  a pod that no longer exists and asserts that panel shows the existing not-found state, without
  erroring, closing, or affecting the rest of the restored layout.
- [ ] 5.3 Confirm an unrecognized/future panel kind in a saved layout's dock JSON goes through
  `ui::unrestored::restore_with` exactly as the automatic restore's own unknown-panel case does.
  Verify: a test loads a `SavedLayout` fixture containing one panel of a synthetic unrecognized kind
  and asserts every other panel restores while that slot shows an `UnrestoredPanel`.

## 6. Settings "Layouts" section

- [ ] 6.1 Add `Section::Layouts` to `ui/settings.rs` (sidebar button, `button_id`, `label_selector`,
  a new `layouts_focus: FocusHandle` that `show` focuses) and a new `app/src/ui/settings/layouts.rs`
  module listing every saved layout via `config::saved_layouts::load_all`, following `panels.rs`'s
  placement (design.md D7). Verify: a render test shows the Settings window, switches to Layouts, and
  asserts two fixture saved layouts are listed by name; an empty store shows an explicit "no saved
  layouts" message.
- [ ] 6.2 Register `settings.show_layouts` (title "Settings: Show Layouts", no default binding,
  context `SettingsWindow`, menu `None`), matching `settings.show_panels`'s existing pattern. Verify:
  a test dispatches the action and asserts `SettingsWindow::section()` becomes `Section::Layouts`.
- [ ] 6.3 Add a per-row Remove control (an icon button with a tooltip, per `icon-buttons.md`, a tab
  stop reachable by Tab and activated by Enter/Space) that calls `confirm_dialog::open` with
  `Severity::Irreversible` (design.md D6), removing the saved layout via `config::saved_layouts::
  remove` on confirm. Verify: a keystroke test Tabs to a row's Remove control, activates it, asserts
  the confirmation opens with Cancel focused, presses Enter, and asserts the layout is NOT removed; a
  second test activates the confirm control and asserts the layout is gone from the section and from
  `layouts/`.
- [ ] 6.4 Confirm the Layouts section is reachable and operable end-to-end by keyboard: Tab from the
  sidebar into the list, through each row's Remove control, matching the other Settings sections'
  tab order conventions. Verify: a `VisualTestContext::simulate_keystrokes` test Tabs from the sidebar
  button through the section without using the mouse and removes a layout entirely by keyboard.

## 7. Command palette, keymap, and menu integration

- [ ] 7.1 Run `keymap::conflicts` against the full registry with the seven new commands from
  design.md D4 registered, confirming none of `cmd-shift-s`, `cmd-shift-o`, `enter`,
  `secondary-enter`, `r`, `backspace` (the last four scoped to `SavedLayoutsPicker`), or
  `settings.show_layouts`'s empty binding collide with an existing one. Verify: the existing `keymap`
  conflict test suite passes with the new commands included; add a fixture case if the suite does not
  already parameterize over the full live registry.
- [ ] 7.2 Confirm all seven commands appear in the command palette with correct context gating (the
  four picker-scoped ones only listed while the picker has focus, `settings.show_layouts` only while
  the Settings window has focus) and that `layouts.save` and `layouts.manage` each have a Window-menu
  entry. Verify: a palette test asserts item counts/content change as the active `KeyContext` stack
  gains and loses `SavedLayoutsPicker` and `SettingsWindow`; a menu-building test asserts both
  top-level commands appear under `MenuSlot::Window`.
- [ ] 7.3 Add hint-row key display (`Kbd::binding_for_action`) for the picker's row actions (load with
  Add, load with Replace, rename, delete) and the Settings section's Remove control, matching the
  existing Pods-panel and `ui/picker_keys.rs` convention. Verify: a render test asserts the picker and
  the Layouts section show the live (possibly user-overridden) keys, not the hardcoded defaults, by
  loading a `keymap.toml` fixture that overrides one of them.

## 8. End-to-end round trip

- [ ] 8.1 Add an integration-style test: save a layout from a window with two panels across two
  contexts, reload `SavedLayout`s from disk (simulating a restart), open the picker, load it with
  Replace into the same window, and assert the window's panels, Resource panel width, and bounds
  match what was saved. Verify: this test passes without a live cluster (mocked kube client /
  fixtures, per project convention).
- [ ] 8.2 Add an integration-style test for Add: open a window with one panel, load a different saved
  layout with Add, and assert the window now shows both the original panel and the saved layout's
  panels, with no duplicate for any matching content key. Verify: same fixture-based approach as 8.1.
- [ ] 8.3 Update `docs/architecture.md`'s persistence section to list the `layouts/` folder alongside
  `workspace.toml` and `dock-layouts.json`, keeping it in sync with `openspec/config.yaml`'s
  `context:` block per this repo's stated convention. Verify: a manual diff of the two confirms they
  describe the same persisted state.
