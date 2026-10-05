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

- [x] 4.1 Add a `CloseGroup` action that closes every panel in the focused group, reusing the
      existing per-panel confirmation path and batching any confirmations into a single dialog
      listing the affected panels, and verify with integration tests covering a group with no
      confirmable panels (closes immediately), a group with confirmable panels where the user
      confirms (all close), and where the user cancels (none close)
- [x] 4.2 Register `CloseGroup` in the command registry with a default keybinding, title, and
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

- [ ] 6.1 Confirm the keybindings editor lists all twelve new commands (split, move, merge) and
      `CloseGroup` as rebindable entries, and verify by opening the editor in a manual test pass
- [x] 6.2 Update `tasks.md` progress and any relevant doc comments once all tests above pass

## Notes

- Implemented in pilgrimagesoftware/Fernrohr-App#123, stacked on App#122
  (`k9s-remaining-keybindings`): Close Group's confirmation needs panels with something to lose,
  and the first such panels - a running shell, an unsaved YAML edit - arrive there.
- 6.1 is left for a person: the keybindings editor lists all thirteen commands (an automated test,
  `every_arrange_command_has_an_editor_row`, checks the rows), but the task asks for a look in a
  manual pass.
- Default keys: split `cmd-k <arrow>`, move `cmd-alt-<arrow>`, merge `cmd-alt-shift-<arrow>`, close
  group `cmd-k w` - none collides with an existing binding.
- "Adjacent" is computed from the pane tree's own split proportions rather than captured paint
  bounds: the dock exposes no per-group bounds, and the tree's proportions are what it lays out.
  An exact tie (one pane beside two stacked ones) is no neighbour, per the design's risk note.
- The design's "reuse the per-panel confirmation path" has nothing to reuse: closing one panel
  never confirms today. Close Group asks (one dialog, listing what's lost) only for panels that
  report something to lose - `close_warning` on the shell and object panels. Closing such a panel
  by its own tab still doesn't ask; worth deciding separately whether it should.
- The commands are scoped to a new `Dock` key context around the dock area, the "a panel group has
  focus" predicate the design asks for: off while a dialog or the Resource panel has focus.
