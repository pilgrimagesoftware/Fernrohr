# Proposal

## Why

Panel tabs currently expose no right-click menu. Closing a tab other than the focused one, closing
a whole group of tabs at once, or moving a tab into its own window all require either dragging or
hunting through the command palette for the right action name. A tab context menu puts the small,
tab-scoped set of actions a user reaches for constantly (close this, close the rest, split this
out) one right-click away, matching the convention every tabbed editor and browser already uses.

## What Changes

- Right-clicking a panel's tab (or invoking the equivalent keyboard menu action on the focused
  tab) opens a context menu scoped to that tab, using gpui-kit's `ContextMenu`/`PopupMenu`.
- Menu items, in order:
  - Close Tab
  - Close Other Tabs (disabled when it is the only tab in its group)
  - Close Tabs to the Right (disabled when already rightmost)
  - Close All Tabs (closes the whole group)
  - Move to New Window (splits the tab out into its own window)
  - Pin Tab / Unpin Tab (pinned tabs sort first and are excluded from the "close others"/"close
    all" actions)
- Every menu item is a thin wrapper around an existing or newly-registered command-registry
  action (`ClosePanel`, `ClosePanelGroup`, new `CloseOtherPanels`, `ClosePanelsToRight`,
  `MovePanelToNewWindow`, `PinPanel`/`UnpinPanel`), so each action also appears in the command
  palette and is independently keybindable - the menu is one more surface onto commands that
  already exist or are added by this change, not a parallel action path. `ClosePanelGroup` is
  neither existing nor this change's to define: `panel-move-keybindings` introduces it as its own
  group-close action (`CloseGroup` in that change's own design/tasks text - same command, two
  names in flight; reconcile on whichever lands first), so "Close All Tabs" here depends on that
  change landing first and calls its command rather than adding a second one.
- Any close action that would disconnect a context's last tunnel reuses the existing
  tunnel-aware confirmation dialog (same path as `Cmd-W` and group-close today).
- The menu is reachable and fully operable from the keyboard: a keybound action opens it for the
  focused tab, arrow keys move between items, and Enter activates the highlighted one.

## Capabilities

### New Capabilities

- `panel-tab-menu`: the tab-scoped context menu itself - what it lists, when items are enabled or
  disabled, and how it is reached by mouse and keyboard.

### Modified Capabilities

(none - pinning and the new close variants are introduced as part of `panel-tab-menu` rather than
as changes to `app-shell`'s existing workspace requirements)

## Impact

- `App/src/ui/` dock/workspace layer wrapping gpui-kit's `TabPanel`/tab strip, where the
  right-click handler and menu-open keybinding attach.
- `App/src/command.rs` central command registry: new action ids for `CloseOtherPanels`,
  `ClosePanelsToRight`, `MovePanelToNewWindow`, `PinPanel`, `UnpinPanel`.
- `keymap.toml` defaults: one new binding to open the focused tab's context menu.
- Panel/workspace persistence (`workspace.toml`): pinned state needs to round-trip through save
  and restore.
- gpui-kit's `menu::context_menu` / `menu::popup_menu` (0.7.0, used from crates.io).
