# Tasks

## 1. Saved-layouts store

- [ ] 1.1 Add `SavedLayout` and `SavedLayouts { version, layouts: Vec<SavedLayout> }` (per design.md
  D2) next to `app/src/config/dock_layouts.rs`, with `load()`/`save()` at
  `state_dir()/saved-layouts.json` following `persist.rs`'s existing convention: a parse failure
  leaves the file untouched and falls back to an empty list. Verify with unit tests: round-trip
  save-then-load of a `SavedLayouts` with one entry; a corrupt file on disk loads as empty without
  being overwritten.
- [ ] 1.2 Add `SavedLayouts::insert_or_overwrite(name, ...) -> OverwroteExisting(bool)`,
  `rename(old, new) -> Result<(), NameTaken>`, and `remove(name)`. Verify with unit tests: inserting
  a new name reports no overwrite; inserting an existing name reports an overwrite and replaces the
  entry; renaming to a name already in use returns `NameTaken` and leaves both entries unchanged;
  removing an absent name is a no-op.

## 2. Save Panel Layout command

- [ ] 2.1 Register the `layouts.save` command (id, title "Save Panel Layout…", default binding
  `cmd-shift-s`, context `Workspace`, menu `MenuSlot::Window`) per design.md D5, wired to a naming
  dialog (text input, Tab-reachable, Enter confirms, Escape cancels). Verify: a
  `VisualTestContext::simulate_keystrokes` test opens the dialog via the default binding and
  confirms it appears; a render test confirms it does not appear while a window is in cluster-
  picker mode (no `Workspace` context active).
- [ ] 2.2 Wire the dialog's confirm to capture the current window's dock (`DockArea::dump`),
  Resource panel width/visibility, contexts, and window bounds into a `SavedLayout`, calling
  `insert_or_overwrite`. Verify: a test saves a window with two panels and asserts the resulting
  `SavedLayout`'s dock JSON contains both panels' content keys and the window's current bounds.
- [ ] 2.3 Add the overwrite confirmation step when the entered name already exists. Verify: a
  keystroke test enters an existing name, asserts a confirmation prompt appears, and that declining
  it leaves the existing saved layout's `updated_at` unchanged.
- [ ] 2.4 Add the structural secret-safety regression test from design.md D6: reveal a `SecretValue`
  in a panel, save a layout, and assert `format!("{:?}", saved_layout)` (and the serialized JSON)
  contains no fixture secret value.

## 3. Saved Layouts picker (list, rename, delete)

- [ ] 3.1 Add a `SavedLayoutsPicker` view under `app/src/ui/picker/`, following `ClusterPicker`'s
  structure (state/interaction/layout/rows/render split, `window.last_input_was_keyboard()` to
  distinguish hover from keyboard selection), listing every `SavedLayout` by name. Verify: a render
  test with a fixture `SavedLayouts` of two entries shows both names; an empty `SavedLayouts` shows
  an explicit "no saved layouts" message rather than a blank list.
- [ ] 3.2 Register `layouts.manage` (title "Saved Layouts…", default binding `cmd-shift-o`, context
  `None`, menu `MenuSlot::Window`) opening the picker, available from both `Workspace` and
  cluster-picker window modes. Verify: a keystroke test opens the picker via the default binding
  from a window in cluster-picker mode.
- [ ] 3.3 Confirm hover never moves the keyboard-selected row: wire row hover through
  `last_input_was_keyboard()` the same way `ClusterPicker` does, not through the underlying
  `Command` widget's own hover-selects behavior. Verify: a test simulates arrow-key navigation to
  select row 2, then simulates a mouse-hover event over row 1, and asserts row 2 is still selected.
- [ ] 3.4 Register `saved_layouts.rename_selected` (default binding `r`, context
  `SavedLayoutsPicker`) opening an inline rename field; reject and show the existing "name already
  in use" case from 1.2. Verify: a keystroke test renames the selected row by pressing `r`, typing
  a new name, and Enter, then asserts the picker lists the new name; a second test attempts a
  colliding rename and asserts both original names remain.
- [ ] 3.5 Register `saved_layouts.delete_selected` (default binding `backspace`, context
  `SavedLayoutsPicker`) with inline (not nested-modal) confirmation per design.md D5. Verify: a
  keystroke test presses backspace, confirms, and asserts the row is gone from the picker and from
  `saved-layouts.json` on disk; a second test presses backspace and then Escape/cancel and asserts
  the row remains.

## 4. Restoring a saved layout

- [ ] 4.1 Register `saved_layouts.restore_in_window` (default binding `enter`, context
  `SavedLayoutsPicker`): replaces the current window's dock (via `DockArea::load` from the saved
  `DockAreaState`), Resource panel width/visibility, and window bounds. Verify: a test opens the
  picker from a window with one arrangement, restores a different saved layout, and asserts the
  window's dock, Resource panel width, and bounds now match the saved layout, not the prior state.
- [ ] 4.2 Register `saved_layouts.restore_in_new_window` (default binding `cmd-enter`, context
  `SavedLayoutsPicker`): opens a new window at the saved bounds (cascaded per design.md D5) with the
  saved dock, leaving the originating window untouched. Verify: a test restores into a new window
  and asserts two windows now exist, the original unchanged and the new one matching the saved
  layout.
- [ ] 4.3 Reuse `restored_panel_keys()`/`PanelKey` decoding (per design.md D3) for a saved layout's
  dock JSON rather than adding a parallel decoder. Verify: a shared-fixture test asserts the same
  `PanelKey` values decode whether the source is `dock-layouts.json` or a `SavedLayout.dock`.

## 5. Missing context, namespace, and object handling on restore

- [ ] 5.1 Classify each restored panel's context per design.md D4: connected already, known-but-
  disconnected (connect on demand), or unknown to the current kubeconfig (placeholder). Verify: a
  test restores a layout referencing a disconnected-but-known context and asserts a connection
  attempt starts automatically; a test restores a layout referencing a context absent from a fixture
  kubeconfig and asserts that panel renders as a placeholder offering to pick a replacement context,
  while the layout's other panels restore normally.
- [ ] 5.2 Add the namespace/object-not-found placeholder for restored panels, following
  `pod-detail`'s existing "pod no longer exists" pattern. Verify: a test restores a layout whose pod
  detail panel points at a pod that no longer exists and asserts that panel shows the not-found
  placeholder with a retry action, without erroring or closing, and without affecting the rest of
  the restored layout.
- [ ] 5.3 Handle an unknown/future panel kind in a saved layout's dock JSON the same way the
  automatic restore already does (skip with a placeholder, not a failed restore). Verify: a test
  restores a `SavedLayout` fixture containing one panel of a synthetic unrecognized kind and asserts
  every other panel restores while that slot shows "not restorable in this version".

## 6. Command palette, keymap, and menu integration

- [ ] 6.1 Run `keymap::conflicts` against the full registry with the six new commands from design.md
  D5 registered, confirming none of `cmd-shift-s`, `cmd-shift-o`, `enter`/`cmd-enter`/`r`/`backspace`
  (scoped to `SavedLayoutsPicker`) collide with an existing binding. Verify: the existing
  `keymap` conflict test suite passes with the new commands included; add a fixture case if the
  suite does not already parameterize over the full live registry.
- [ ] 6.2 Confirm all six commands appear in the command palette with correct context gating (the
  four picker-scoped ones only listed while the picker has focus) and that `layouts.save` and
  `layouts.manage` each have a Window-menu entry. Verify: a palette test asserts item counts/content
  change as the active `KeyContext` stack gains and loses `SavedLayoutsPicker`; a menu-building test
  asserts both top-level commands appear under `MenuSlot::Window`.
- [ ] 6.3 Add hint-row key display (`Kbd::binding_for_action`) for the picker's row actions,
  matching the existing Pods-panel and `ui/picker_keys.rs` convention. Verify: a render test asserts
  the picker shows the live (possibly user-overridden) keys for restore/rename/delete, not the
  hardcoded defaults, by loading a `keymap.toml` fixture that overrides one of them.

## 7. End-to-end round trip

- [ ] 7.1 Add an integration-style test: save a layout from a window with two panels across two
  contexts, close the window, restart the in-test app state (reload `SavedLayouts` from disk), open
  the picker, restore in a new window, and assert the new window's panels, Resource panel width, and
  bounds match what was saved. Verify: this test passes without a live cluster (mocked kube client /
  fixtures, per project convention).
- [ ] 7.2 Update `docs/architecture.md`'s persistence section to list `saved-layouts.json` alongside
  `workspace.toml` and `dock-layouts.json`, keeping it in sync with `openspec/config.yaml`'s
  `context:` block per this repo's stated convention. Verify: a manual diff of the two confirms they
  describe the same three files.
