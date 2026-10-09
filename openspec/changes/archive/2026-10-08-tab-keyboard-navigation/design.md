# Design

## Context

See `proposal.md` for why. What gpui-component / gpui-base 0.6.6 offer, and what
`panel-focus-navigation` already built:

- **Reading a group:** each group is a public `PaneRef::Tabs { panels, active_ix }` in a region's
  `PaneTree`. `PaneTree::find_panel_node(panel)` finds a panel's group.
- **Switching a tab:** `DockArea::select_panel(id)` is "the one way to select a panel by
  identity", but it moves no keyboard focus. The group entities themselves are private to the
  dock, so there's no `TabGroup::select_tab` to call.
- **No key contexts in the dock,** so a command can't be scoped "to a tab group". The commands
  are global and `MainWindow` runs them, like `panel.focus_next`.
- **Reusable pieces:** `ui/panel/focus.rs` already has the region walk (`dock_stops`, whose stops
  are exactly the groups in panel-focus order) and `step` (wrapping index arithmetic).
- **Nothing in gpui-component binds** `cmd-shift-]` / `cmd-shift-[` or `ctrl-1`…`ctrl-9`. The
  only modified-bracket bindings it has are the text input's `cmd-]` / `cmd-[` (Indent/Outdent,
  macOS) and `ctrl-]` / `ctrl-[` (elsewhere). None of the app's registered commands use these keys.

## Goals / Non-Goals

**Goals:**
- Every tab of every group is reachable from the keyboard.
- One definition of "the focused tab group", shared with `per-tab-close-button`'s `Cmd-W`.

**Non-Goals:**
- Moving a tab to another group, or reordering tabs, from the keyboard.
- A per-tab close button: `per-tab-close-button`'s Options A/B stay deferred.
- Most-recently-used tab order (Ctrl-Tab style). Tab order is the strip's left-to-right order.

## Decisions

### The focused tab group

The focused group is the group whose displayed panel `contains_focused`. When focus is outside
every group (the Resource panel, or nothing), it's the group of the first panel-focus stop
(`dock_stops(..)[0]`). A dock with no panels has no focused group, and the commands do nothing.

*Why a fallback, rather than doing nothing outside the dock:* with focus in the Resource panel,
"next tab" (and `Cmd-W`) should still act on the tabs the user can see. The first stop is the
same group `cmd-]` would move to, so the fallback is predictable. *Alternative:* remembering the
last-focused group. That needs history that has to survive panels closing, for little gain once
`cmd-]` exists.

### Switching and focusing

The next index comes from `focus::step` over the group's `panels`, from `active_ix`. It wraps, and
one tab stays put. "Select tab N" indexes directly, with 9 meaning the last tab and a position
past the end doing nothing. Either way the result is `select_panel(target)` followed by focusing
the target's handle. That's the same pair `open_target_with_view` uses to re-show a panel, so a
tab shown from the keyboard behaves like one asked for by name.

### Commands and keys

| id | title | default |
|---|---|---|
| `tab.next` | Next Tab | `cmd-shift-]` |
| `tab.previous` | Previous Tab | `cmd-shift-[` |
| `tab.select_1` … `tab.select_9` | Select Tab 1 … Select Tab 9 (9 = last) | `ctrl-1` … `ctrl-9` |

All global, like the `panel.*` commands. Next/Previous go in the Navigate menu. Select Tab N stay
out of the menu (nine near-identical items) but are in the palette. `cmd` is GPUI's platform key,
as for the app's other commands.

*Alternatives:*
- `ctrl-tab` / `ctrl-shift-tab`: free too, but on macOS apps they usually mean
  most-recently-used switching, which this isn't.
- `cmd-1`…`cmd-9`: taken by the show-panel commands (`cmd-1` Pods, `cmd-2` Logs).
- `alt-digit`: types characters (`¡`, `™`…) on macOS keyboards, so it would compete with text
  fields.
- `ctrl-digit` has one known cost, see Risks.

A registry-wide test asserts no two commands share a default. A test through the Resource filter
field confirms a focused text input passes the keys through. The input binds neither shifted
bracket nor `ctrl-digit`, so this is a guard rather than a known interaction.

### Relation to `per-tab-close-button`'s `Cmd-W`

`Cmd-W` uses the same focused-group helper. If there is a focused group, it focuses that group's
displayed tab and then dispatches `ClosePanel`. `ClosePanel` is handled by the tab group, so it
only fires with focus inside one; #27's design dispatched it whenever the center dock was
non-empty, which does nothing while the Resource panel has focus. If there's no focused group
(the dock is empty), `Cmd-W` takes #27's close-window path, with its tunnel confirmation,
unchanged. `per-tab-close-button/design.md` gets a note pointing here, and both land in one App
task (`tab-keyboard-app`).

## Risks / Trade-offs

- **[macOS "Switch to Desktop N" uses `ctrl-1`…`ctrl-9` when the user enables it in System
  Settings. It's off by default.]** When on, the OS takes the key first. Rebind in `keymap.toml`.
  `tab.select_*` ids are stable for that.
- **[Nine palette entries for Select Tab N.]** They're grouped by a shared "Select Tab" prefix, so
  typing "tab" finds them together. The cost is a few extra rows.
- **[The fallback group is only a guess at intent when focus is outside the dock.]** It's the
  same group `cmd-]` goes to, and the tab underline shows where focus landed. For `Cmd-W`, a
  wrong guess closes a tab the user has to re-open, so the `tab-keyboard-app` tests pin which tab
  it closes with focus in the Resource panel.
- **[The App side waits on split-shell.]** `MainWindow`'s module layout is changing under Builder
  2's split-shell, and #27's `Cmd-W` handling lives there. No App code until it merges.
