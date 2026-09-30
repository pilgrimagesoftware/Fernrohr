# Design

## Context

See `proposal.md` for why. Facts about gpui-component / gpui-base 0.6.6 that shape the approach:

- **The dock has no panel-to-panel focus movement.** The only dock actions are `ToggleZoom` and
  `ClosePanel`. Nothing tracks "the focused panel" at the `DockArea` level.
- **The layout is readable.** `DockArea::layout(placement) -> Option<&PaneTree>` gives each
  region's tree (`Center`, `Left`, `Right`, `Bottom`). `PaneTree::root()` and
  `PaneNode::walk()` do a pre-order walk; that order is on-screen order within a split. Each
  tab group is a `PaneRef::Tabs { panels, active_ix }`. `DockArea::panel(id)` returns the
  panel's view, and its `focus_handle(cx)` is public.
- **Nothing in the dock moves keyboard focus.** `add_panel_view` makes the new panel the active
  tab of its group, and `select_panel` switches tabs, but neither focuses anything.
  `shell::open_target_with_view` calls one or the other and stops there.
- **Hidden regions still appear in the layout.** A collapsed dock's `PaneTree` still lists its
  panels (`is_dock_open` says whether it's shown). While a group is zoomed (`is_zoomed`,
  `zoomed_group`), the `DockArea` draws only that group.
- **The Resource panel is outside the dock.** It's a split beside the `DockArea`, reached today by
  `resource.focus` (`cmd-0`, handled by `MainWindow`).
- There are no panel bounds to read, so directional (left/right/up/down) focus would need new
  per-group bounds tracking.

## Goals / Non-Goals

**Goals:**
- Next/previous-panel commands that reach every panel the user can see, including the Resource
  panel, in a stable, predictable order.
- Opening or re-showing a panel from any route focuses it.

**Non-Goals:**
- **Directional focus (left/right/up/down).** It needs on-screen bounds per group, which the dock
  doesn't expose. Cyclic order reaches every visible panel without it. Can follow later.
- **Switching tabs within a group from the keyboard.** Next/previous visits each *visible* panel,
  that is each group's displayed tab. Reaching a hidden tab in the same group is a separate
  command (e.g. next/previous tab) for a separate change.
- Restoring which panel had focus across relaunches.

## Decisions

### Focus stops: the Resource panel, then each group's displayed tab, in reading order

The stops are, in order:
1. The Resource panel, when the window shows one.
2. The dock regions in `Left, Center, Right, Bottom` order, skipping any dock where
   `!is_dock_open`. Within a region, pre-order over its tree, taking each tab group's
   displayed panel (`panels[active_ix]`) and skipping panels whose `visible(cx)` is false.

While a group is zoomed, the dock contributes only that group's displayed panel, since nothing
else in the dock is drawn. The Resource panel sits outside the `DockArea`, so it stays a stop.

*Why this order:* it's left-to-right, then the bottom strip. That matches how the window reads,
and it's stable because it comes from the layout, not from which panel was focused last.
*Alternatives:* most-recently-focused order (Alt-Tab style) is less predictable for "previous",
and needs history that has to survive panels closing. Directional order needs bounds (see
Non-Goals).

### Two pure functions carry the logic

- `focus_stops(area, resource, cx) -> Vec<Stop>` builds the ordered list described above.
- `step(len, current: Option<usize>, direction) -> Option<usize>` picks the next index. It wraps
  at both ends, and when nothing is focused, `Next` gives the first stop and `Previous` the last.

"Current" is the stop whose focus handle `contains_focused`, the same test the tab underline
uses. So focus inside a panel's table row still counts as being on that panel. Keeping `step`
pure means wrap-around and no-focus cases get plain unit tests, and the stop order gets tests
over built layouts, with no window needed.

### Commands: registered, global, in the Navigate menu

`panel.focus_next` "Focus Next Panel" and `panel.focus_previous` "Focus Previous Panel", with
`context: None`. They follow `resource.focus`: `MainWindow` handles both, because it owns the
Resource panel and the `DockArea`, and both go in `MenuSlot::Navigate`.

Default bindings: **`cmd-]`** next, **`cmd-[`** previous (`cmd` is GPUI's platform key, the same
as `cmd-n` and `cmd-0`). None of the app's registered commands use them. gpui-component's text
input binds the same keys to Indent/Outdent on macOS (`ctrl-]` / `ctrl-[` elsewhere), but only a
multi-line input handles those actions. GPUI tries each matched binding in turn until one is
handled, so in the app's single-line filter fields the key falls through to the panel command.
A test pins this.
*Alternatives:*
- `F6` / `shift-F6` is the conventional "cycle panes" key on Windows and Linux, but on a Mac
  keyboard it needs `fn`.
- `ctrl-tab` suggests "next tab in this group", which these commands deliberately don't do.
- ``cmd-` `` is macOS's own window cycling.

A test builds the full app registry and asserts no other command has the same default in an
overlapping context.

### An opened panel is focused in one place: `open_target_with_view`

After both arms (the existing panel re-selected with `select_panel`, or a new one added with
`add_panel_view`), focus `area.panel(id)`'s handle. One site covers every route that opens a panel
through the window: Resource rows, `nav.*` commands, and panel shortcuts that dispatch `nav`
actions. That's better than repeating it in each `nav::add_panel` arm, which re-shows would miss.

## Risks / Trade-offs

- **[Opening a panel moves focus away from the panel that opened it.]** For example, `l` in Pods
  now leaves focus in the Logs panel. This is intended (see the spec). `cmd-[` goes back one stop,
  and the tab underline shows where focus went.
- **[`open_target_with_view` is being edited by other in-flight work (typed references / object
  viewer).]** The focus call is a few lines at the end of the function. Whoever lands second
  rebases. Coordinate through Fernrohr Master.
- **[In the default layout, every opened panel is a tab in the one center group.]** Next/previous
  therefore steps between the Resource panel and the displayed tab until the user splits. The other
  tabs stay keyboard-reachable by asking for them again (`cmd-1`, `cmd-2`, Enter on a Resource row,
  `d`/`l`/`y`), which now focuses them. A next/previous-*tab* command is the natural follow-up.
- **[A layout change mid-cycle (a panel closed) changes the order.]** Stops are rebuilt on every
  invocation from the live layout, so there's no stale index. A closed panel simply isn't a stop.
- **[`cmd-[` / `cmd-]` inside a text input.]** Some editors use them for indent/outdent. Fernrohr's
  inputs (filter fields) are single-line, so there's nothing to indent, and the global binding is
  the more useful meaning. If that changes, `keymap.toml` can rebind.

## Open Questions

- Should directional focus (left/right/up/down) follow? It depends on usage feedback once cyclic
  focus exists. It doesn't change this change's specs or tasks.
