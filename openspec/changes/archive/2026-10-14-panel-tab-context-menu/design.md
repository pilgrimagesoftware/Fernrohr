# Design

## Context

`App/src/ui/panel/tabs.rs` already owns "the focused tab group" and per-tab targeting
(`TabGroup`, `target_index`, `focused_group`, `group_of`) for Next/Previous/Select-N-Tab and the
dock's zoom toggle. `util::shell` runs tab-scoped commands (e.g. `Cmd-W`'s close) against that
focused group and owns the tunnel-aware disconnect confirmation this change reuses. Every
user-facing action is a `Command` in the central registry (`src/command.rs`); the palette, menu
bar, and `keymap::bindings` all derive from it - no raw `KeyBinding`s. gpui-kit 0.7 (crates.io)
ships `component::menu::{context_menu, popup_menu}`, already a project dependency.

See proposal.md for why a tab context menu is being added; see
`specs/panel-tab-menu/spec.md` for the exact requirements.

## Goals / Non-Goals

**Goals:**
- Reuse `tabs.rs`'s existing group/tab model instead of introducing a second notion of "the
  target tab" - the menu always targets the tab it was opened for, not "the focused tab", since a
  right-click can target a non-focused tab.
- Every menu item is a thin dispatch to a registered `Command`, so nothing in the menu bypasses
  the palette/keymap.

**Non-Goals:**
- Dragging tabs between groups, splitting panes, or any other dock-layout gesture - out of scope,
  unchanged by this design.
- A generic "right-click anywhere opens a menu" mechanism - this covers tab menus only.

## Decisions

**Target resolution: by `PanelId`, not "the focused group".** The menu is opened for a specific
`PanelId` (the tab clicked, or - for the keyboard path - the focused group's active panel via
`tabs::focused_group`). Every action the menu runs resolves its target panel/group fresh at
invocation time from that `PanelId` via `tabs::group_of`, rather than capturing a `NodeId`/index
at menu-open time, so a tab that moved or closed between menu-open and item-select (e.g. another
keybinding fired while the menu was open) can't make the action act on the wrong tab. Alternative
considered: capture the resolved `TabGroup` snapshot at open time - rejected, since gpui-kit's
popup menu can stay open across other input and a stale index is exactly the kind of bug this
file's existing comments call out avoiding.

**New commands, not menu-only closures.** `CloseOtherPanels`, `ClosePanelsToRight`,
`MovePanelToNewWindow`, `PinPanel`, `UnpinPanel` are registered in `command.rs` like every other
action (id, title, default binding - none, except the menu-open action itself - context, menu
slot). The menu builds its items by looking up these commands' current titles/enabled state
rather than hardcoding strings, so a title rename in the registry doesn't need a second edit in
the menu-building code. This follows the existing pattern (`tab.next`, `panel.toggle_zoom`) rather
than inventing a parallel path.

**`ClosePanelGroup` ("Close All Tabs") is reused, not defined here.** `panel-move-keybindings` is
the change that registers the group-close command - this one's "Close All Tabs" menu item is a
thin wrapper dispatching that existing command, extended (by whichever of the two changes lands
second) to skip pinned tabs. If `panel-tab-context-menu` lands first, its own tasks build the menu
item against `panel-move-keybindings`'s command id as a forward reference and the dependency is
satisfied once that change merges; it does not add a second group-close action of its own.

**Pinned state lives on the panel, not the dock layout.** A new `pinned: bool` persists wherever
panel-local config already round-trips through `workspace.toml` (next to the panel's `ResourceView`
state), not in `DockArea`'s own layout tree - pin/unpin is a property of a panel's identity, and
survives the panel moving between groups (e.g. via "Move to New Window") the same way its filters
and selection do.

**"Move to New Window" reuses the existing new-window-with-panel path.** Whatever currently backs
"open a new window with one panel" (used today for e.g. opening a context's Pods panel in a fresh
window) is the function this action calls after removing the panel from its current group -
no new window-construction code.

## Risks / Trade-offs

[gpui-kit's `PopupMenu`/`ContextMenu` may not expose per-item disabled styling in 0.7.0] →
Verify against the vendored source before implementation; if disabled rendering is missing,
fall back to omitting a disabled item from the list rather than drawing it unstyled, and file
the gap upstream.

[Resolving the target by `PanelId` on every action adds an indirection other tab commands don't
have] → Acceptable: `tabs::group_of` is an existing O(panels) lookup already used for other
by-panel queries, and correctness under a reordered/closed tab matters more than the lookup cost
for a context menu.

[Pinned tabs are a new piece of persisted state] → Add it to the same `workspace.toml` migration
path other panel fields already use (missing field defaults to `false` on load), so existing
saved workspaces open unaffected.
