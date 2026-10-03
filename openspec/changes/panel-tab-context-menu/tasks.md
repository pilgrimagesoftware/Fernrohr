# Tasks

## 1. Pinned panel state

- [ ] 1.1 Add `pinned: bool` to the panel-local state that round-trips through `workspace.toml`,
  defaulting to `false` on load for existing files; verify with a test that an old-format
  workspace file (no `pinned` field) loads with every panel unpinned.
- [ ] 1.2 Add a query for "is this panel pinned" and a setter that flips it, next to the existing
  panel-state accessors; verify with a unit test that pin → save → load round-trips `true`.

## 2. New commands

- [ ] 2.1 Register `CloseOtherPanels` and `ClosePanelsToRight` actions in `command.rs`, each
  resolving their target group via `tabs::group_of(panel_id)` and closing every other/every
  rightward tab through the existing per-tab close path (so the tunnel-disconnect confirmation
  already wired there fires unchanged); verify with tests that each closes the expected tabs and
  leaves the rest, including the already-covered tunnel-confirmation case.
- [ ] 2.2 Register `PinPanel`/`UnpinPanel` actions that toggle the state from Task 1.2 and update
  tab sort order (pinned tabs first within their group); verify with a test that pinning a tab
  moves it before unpinned siblings in `TabGroup::panels` order.
- [ ] 2.3 Register `MovePanelToNewWindow`, removing the panel from its current group and reopening
  it as the sole panel of a newly opened window via the existing new-window-with-panel path,
  preserving its `ResourceView` state; verify with a test that the panel's cluster/kind/
  namespace/filter/selection are unchanged after the move.
- [ ] 2.4 Extend `ClosePanelGroup`'s (or add, if "Close All Tabs" needs its own id) bulk-close path
  to skip pinned tabs; verify with a test that "Close All Tabs" on a group with one pinned tab
  leaves that tab open and closes the rest.
- [ ] 2.5 Add a keybound `OpenTabContextMenu` action, scoped like the other `tab.*` commands, that
  opens the menu from Task 3 for `tabs::focused_group`'s active panel; verify with a
  `simulate_keystrokes` test that invoking it opens the menu for the correct panel.

## 3. Context menu

- [ ] 3.1 Build the tab context menu using gpui-kit's `component::menu::context_menu`/
  `popup_menu`, listing Close Tab, Close Other Tabs, Close Tabs to the Right, Close All Tabs,
  Move to New Window, Pin Tab/Unpin Tab in that order, each item's label and enabled state read
  from its backing `Command`; verify with a test that builds the menu for a sample `TabGroup` and
  asserts the item list, labels, and disabled items match the group's shape (only tab in group;
  rightmost tab; pinned vs unpinned).
- [ ] 3.2 Wire the tab strip's right-click to open this menu for the clicked `PanelId` at the
  pointer location; verify with a UI test (or the project's existing click-simulation pattern)
  that right-clicking a non-focused tab opens a menu targeting that tab, not the focused one.
- [ ] 3.3 Wire menu item selection to dispatch the corresponding `Command`'s action against the
  `PanelId` the menu was opened for, re-resolving the group fresh at selection time (per
  design.md's target-resolution decision) rather than from a captured snapshot; verify with a
  test that closes/reorders tabs while the menu is conceptually open (call the stored `PanelId`
  resolver after mutating the dock) and confirms the action still targets the right panel.

## 4. Integration and documentation

- [ ] 4.1 Add every new command to `keymap.toml`'s defaults (empty binding where none is
  specified in the proposal) and confirm the command palette lists all six actions by running the
  existing palette-listing test (or adding one) against the updated registry.
- [ ] 4.2 Run `cargo fmt && cargo clippy -- -D warnings && cargo test` and fix any failures.
- [ ] 4.3 Update `App/CLAUDE.md` or the relevant doc comment in `ui/panel/tabs.rs` if the module's
  existing "what this file owns" comment no longer matches once the new commands land; verify by
  re-reading the module doc comment against the final file contents.
