# Tasks

## 1. Pinned panel state

- [x] 1.1 Add `pinned: bool` to the panel-local state that round-trips through `workspace.toml`,
  defaulting to `false` on load for existing files; verify with a test that an old-format
  workspace file (no `pinned` field) loads with every panel unpinned.
- [x] 1.2 Add a query for "is this panel pinned" and a setter that flips it, next to the existing
  panel-state accessors; verify with a unit test that pin → save → load round-trips `true`.

## 2. New commands

- [x] 2.1 Register `CloseOtherPanels` and `ClosePanelsToRight` actions in `command.rs`, each
  resolving their target group via `tabs::group_of(panel_id)` and closing every other/every
  rightward tab through the existing per-tab close path (so the tunnel-disconnect confirmation
  already wired there fires unchanged); verify with tests that each closes the expected tabs and
  leaves the rest, including the already-covered tunnel-confirmation case.
- [x] 2.2 Register `PinPanel`/`UnpinPanel` actions that toggle the state from Task 1.2 and update
  tab sort order (pinned tabs first within their group); verify with a test that pinning a tab
  moves it before unpinned siblings in `TabGroup::panels` order.
- [x] 2.3 Register `MovePanelToNewWindow`, removing the panel from its current group and reopening
  it as the sole panel of a newly opened window via the existing new-window-with-panel path,
  preserving its `ResourceView` state; verify with a test that the panel's cluster/kind/
  namespace/filter/selection are unchanged after the move.
- [x] 2.4 Depends on `panel-move-keybindings`, which introduces `ClosePanelGroup`: do not define a
  new group-close command here. Extend its bulk-close path to skip pinned tabs; verify with a test
  that "Close All Tabs" (dispatching `ClosePanelGroup`) on a group with one pinned tab leaves that
  tab open and closes the rest.
- [x] 2.5 Add a keybound `OpenTabContextMenu` action, scoped like the other `tab.*` commands, that
  opens the menu from Task 3 for `tabs::focused_group`'s active panel; verify with a
  `simulate_keystrokes` test that invoking it opens the menu for the correct panel.

## 3. Context menu

- [x] 3.1 Build the tab context menu using gpui-kit's `component::menu::context_menu`/
  `popup_menu`, listing Close Tab, Close Other Tabs, Close Tabs to the Right, Close All Tabs,
  Move to New Window, Pin Tab/Unpin Tab in that order, each item's label and enabled state read
  from its backing `Command`; verify with a test that builds the menu for a sample `TabGroup` and
  asserts the item list, labels, and disabled items match the group's shape (only tab in group;
  rightmost tab; pinned vs unpinned).
- [x] 3.2 Wire the tab strip's right-click to open this menu for the clicked `PanelId` at the
  pointer location; verify with a UI test (or the project's existing click-simulation pattern)
  that right-clicking a non-focused tab opens a menu targeting that tab, not the focused one.
- [x] 3.3 Wire menu item selection to dispatch the corresponding `Command`'s action against the
  `PanelId` the menu was opened for, re-resolving the group fresh at selection time (per
  design.md's target-resolution decision) rather than from a captured snapshot; verify with a
  test that closes/reorders tabs while the menu is conceptually open (call the stored `PanelId`
  resolver after mutating the dock) and confirms the action still targets the right panel.

## 4. Integration and documentation

- [x] 4.1 Add every new command to `keymap.toml`'s defaults (empty binding where none is
  specified in the proposal) and confirm the command palette lists all six actions by running the
  existing palette-listing test (or adding one) against the updated registry.
- [x] 4.2 Run `cargo fmt && cargo clippy -- -D warnings && cargo test` and fix any failures.
- [x] 4.3 Update `App/CLAUDE.md` or the relevant doc comment in `ui/panel/tabs.rs` if the module's
  existing "what this file owns" comment no longer matches once the new commands land; verify by
  re-reading the module doc comment against the final file contents.
- [x] 4.4 Check manually in the app: right-click a background tab and run each item, each acting
  on that tab; open the focused tab's menu with `cmd-k m` and use it by keyboard; pin two tabs,
  restart, and see both still pinned and first; move a filtered, namespace-scoped list tab to a new
  window and see its filter, scope and pin carried over; run Close Other Tabs over an Exec tab and
  see it ask once. Record no cluster-identifying details.

## Implementation notes (Fernrohr-App#280)

- 1.1/1.2: `pinned` lives on `PanelScope` (not in `PanelKey`) and round-trips in each panel's dock
  state through `panel_title::with_saved_scope` / `PanelScope::restored_from`, beside
  `open-panel-new-instance`'s instance number - the dock state (`dock-layouts.json`, saved layouts)
  rather than `workspace.toml`'s window fields, since that is where panel-local state already
  persists. A missing field reads unpinned.
- 2.1/2.4: the bulk closes share `arrange::close_confirming` (generalised from `close_group`'s one
  combined confirmation); `ClosePanelGroup` itself now skips pinned tabs, so Close All Tabs is that
  command, unchanged in id.
- 2.3: Move to New Window dumps the tab's dock state, closes it, and `load_replace`s a one-tab
  layout into a new window on its context. With duplicated panels (`open-panel-new-instance`) the
  moved tab drops its instance number - it's the new window's first.
- 2.5: `tab.menu` (`cmd-k m`) is scoped `Dock && !Input` like the other `cmd-k` chords, not global
  as the other `tab.*` commands are, so a text field keeps `cmd-k`.
- 3.3: an item hands the command its tab (`TabMenuState::target`) and dispatches from the window
  root; the command takes it in place of the focused tab and resolves the group afresh. A tab not
  yet drawn isn't on the focus path, so focusing it and dispatching from it doesn't reach the
  window.
- Deviation, spec "Close actions confirm when a tunnel would disconnect": closing a tab never
  disconnects a tunnel - the window keeps its contexts with an empty dock (`tab-close-buttons` 4.1),
  and only Close Window and Disconnect release one, both of which already confirm. The menu's
  closes reuse the shell / unsaved-edit confirmation instead.
- Deviation, "Items reflect command-registry state": every label is its command's title except
  Close All Tabs, which runs `panel.close_group` ("Panels: Close Group") under the spec's label.
- No pin icon on the tab (gpui-component has none); pinned state shows in tab order and in the
  menu's Unpin Tab.
