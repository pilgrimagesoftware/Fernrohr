# Tasks

## 1. Group adjacency helper

- [ ] 1.1 Implement a function that, given the focused group and a direction, finds the nearest
      adjacent group in the dock's split tree using rendered bounds, returning `None` when no
      group lies in that direction, and verify with unit tests covering a simple two-pane split,
      an L-shaped three-pane layout, and a single-pane workspace (expect `None`)

## 2. Split-group actions

- [ ] 2.1 Add `SplitGroupLeft`/`Right`/`Up`/`Down` actions that split the focused group in the
      given direction, opening a new pane with a copy of the focused panel's panel-opening
      target, and verify with an integration test asserting the new pane exists and has focus
- [ ] 2.2 Register the four split actions in the command registry with default keybindings,
      titles, and the "a panel group has focus" context predicate, and verify they appear in the
      command palette and `keymap.toml` defaults

## 3. Move-panel actions

- [ ] 3.1 Add `MovePanelLeft`/`Right`/`Up`/`Down` actions that move the focused panel into the
      adjacent group in the given direction (no-op if none exists), removing the now-empty
      source group's pane when applicable, and verify with integration tests covering a
      successful move, a move that empties and removes the source pane, and a move with no
      adjacent group (workspace unchanged)
- [ ] 3.2 Register the four move actions in the command registry with default keybindings,
      titles, and context predicate, and verify they appear in the command palette and
      `keymap.toml` defaults

## 4. Close-group action

- [ ] 4.1 Add a `CloseGroup` action that closes every panel in the focused group, reusing the
      existing per-panel confirmation path and batching any confirmations into a single dialog
      listing the affected panels, and verify with integration tests covering a group with no
      confirmable panels (closes immediately), a group with confirmable panels where the user
      confirms (all close), and where the user cancels (none close)
- [ ] 4.2 Register `CloseGroup` in the command registry with a default keybinding, title, and
      context predicate, and verify it appears in the command palette and `keymap.toml` defaults

## 5. Merge-group actions

- [ ] 5.1 Add `MergeGroupLeft`/`Right`/`Up`/`Down` actions that move every panel from the focused
      group into the adjacent group in the given direction in their existing order and remove the
      focused group's pane (no-op if no adjacent group exists), and verify with integration tests
      covering a successful merge and a merge with no adjacent group (workspace unchanged)
- [ ] 5.2 Register the four merge actions in the command registry with default keybindings,
      titles, and context predicate, and verify they appear in the command palette and
      `keymap.toml` defaults

## 6. Keybindings editor and documentation

- [ ] 6.1 Confirm the keybindings editor lists all twelve new commands (split, move, merge) and
      `CloseGroup` as rebindable entries, and verify by opening the editor in a manual test pass
- [ ] 6.2 Update `tasks.md` progress and any relevant doc comments once all tests above pass
