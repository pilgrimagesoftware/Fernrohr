# Proposal

## Why

The menu bar has grown by accretion. The Context menu interleaves tunnel items with context items;
the View menu is an unordered list with no separators; and the Navigate menu carries every
panel-scoped shortcut (Pods describe/logs/YAML, list keys, Logs), which only work in one panel and
make the menu depend on what has focus.

## What Changes

- **Context**: context actions first, then a separator, then every tunnel action together at the
  bottom.
- **View**: items grouped by purpose with separators between groups (appearance, Resource panel,
  panel layout and table columns).
- **Navigate**: global navigation only - moving focus between panels, Focus Resources, and tab
  selection/cycling. Panel-scoped commands leave the menu bar; they stay in the command palette, the
  keymap and each panel's hint row.

## Capabilities

### New Capabilities

(none)

### Modified Capabilities

- `application-menu`: grouping and separator rules for Context and View; Navigate restricted to
  global navigation.

## Impact

- App: menu assignments (`MenuSlot`) on registered commands and the menu builder's grouping and
  separators. No command is removed from the palette or keymap.
