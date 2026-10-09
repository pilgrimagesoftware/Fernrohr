# Tasks

## 1. Group adjacency helper

- [x] 1.1 Implement a function that, given the focused group and a direction, finds the nearest
      adjacent group in the dock's split tree using rendered bounds, returning `None` when no
      group lies in that direction, and verify with unit tests covering a simple two-pane split,
      an L-shaped three-pane layout, and a single-pane workspace (expect `None`)

## 2. Split-group actions

- [x] 2.1 Add `SplitGroupLeft`/`Right`/`Up`/`Down` actions that split the focused group in the
      given direction, opening a new pane with a copy of the focused panel's panel-opening
      target, and verify with an integration test asserting the new pane exists and has focus
- [x] 2.2 Register the four split actions in the command registry with default keybindings,
      titles, and the "a panel group has focus" context predicate, and verify they appear in the
      command palette and `keymap.toml` defaults

## 3. Move-panel actions

- [x] 3.1 Add `MovePanelLeft`/`Right`/`Up`/`Down` actions that move the focused panel into the
      adjacent group in the given direction (no-op if none exists), removing the now-empty
      source group's pane when applicable, and verify with integration tests covering a
      successful move, a move that empties and removes the source pane, and a move with no
      adjacent group (workspace unchanged)
- [x] 3.2 Register the four move actions in the command registry with default keybindings,
      titles, and context predicate, and verify they appear in the command palette and
      `keymap.toml` defaults

## 4. Close-group action

- [x] 4.1 Add a `ClosePanelGroup` action that closes every panel in the focused group, reusing the
      existing per-panel confirmation path and batching any confirmations into a single dialog
      listing the affected panels, and verify with integration tests covering a group with no
      confirmable panels (closes immediately), a group with confirmable panels where the user
      confirms (all close), and where the user cancels (none close)
- [x] 4.2 Register `ClosePanelGroup` in the command registry with a default keybinding, title, and
      context predicate, and verify it appears in the command palette and `keymap.toml` defaults

## 5. Merge-group actions

- [x] 5.1 Add `MergeGroupLeft`/`Right`/`Up`/`Down` actions that move every panel from the focused
      group into the adjacent group in the given direction in their existing order and remove the
      focused group's pane (no-op if no adjacent group exists), and verify with integration tests
      covering a successful merge and a merge with no adjacent group (workspace unchanged)
- [x] 5.2 Register the four merge actions in the command registry with default keybindings,
      titles, and context predicate, and verify they appear in the command palette and
      `keymap.toml` defaults

## 6. Keybindings editor and documentation

- [x] 6.1 Confirm the keybindings editor lists all twelve new commands (split, move, merge) and
      `ClosePanelGroup` as rebindable entries, and verify by opening the editor in a manual test pass
- [x] 6.2 Update `tasks.md` progress and any relevant doc comments once all tests above pass

## Notes

- Implemented in pilgrimagesoftware/Fernrohr-App#123, one commit per section above.
- Close Group's action is `ClosePanelGroup` (id `panel.close_group`), the name
  `panel-tab-context-menu` reuses.
- Default keys, all under the `cmd-k` prefix: split `cmd-k <arrow>`, move `cmd-k shift-<arrow>`,
  merge `cmd-k alt-<arrow>`, close group `cmd-k w`. None collides with an existing binding, and
  `cmd-k` lists all of them together in the pending-chord popover. Move and merge first defaulted
  to `cmd-alt-<arrow>` and `cmd-alt-shift-<arrow>`, but window managers such as Rectangle take those
  globally, so they never reached the app (#138, pilgrimagesoftware/Fernrohr-App#132).
- The commands bind in `Dock && !Input`. `Dock` is a new key context around the dock area: the "a
  panel group has focus" predicate, off while a dialog or the Resource panel has focus. `!Input`
  keeps every arrange key from firing while a text field has focus; a keystroke test covers this.
- "Adjacent" is computed from the pane tree's own split proportions, not from captured paint bounds,
  because the dock exposes no per-group bounds. An exact tie (one pane beside two stacked ones)
  counts as no neighbour, per the design's risk note.
- Close Group reuses the window's close confirmation, now one shared dialog
  (`open_close_confirmation`), that Close Window also uses. It asks once, listing what's lost, only
  for panels that report something to lose (`close_warning`: a running shell, an unsaved YAML edit).
  The tunnel part of that confirmation doesn't apply: tunnel holds belong to the window, so closing
  panels never releases one. Closing such a panel by its own tab still doesn't ask; whether it
  should is a separate decision.

## Manual checks

- [ ] 6.1: open Settings → Keybindings and confirm all thirteen commands are listed and rebindable
      (`every_arrange_command_has_an_editor_row` checks the rows automatically)
- [ ] With a real keyboard, split, move and merge in each direction; the new or moved panel has focus,
      shown on its tab title
- [ ] With a namespace filter or the YAML editor focused, the arrange keys type or do nothing, and the
      layout is unchanged
- [ ] Close a group holding a running shell or an unsaved edit: it asks once, Cancel keeps every
      panel, Close Group closes the whole group
