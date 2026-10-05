# Design

## Context

Every route to a panel already goes through `MainWindow::open_target_in` (`util/shell/open.rs`).
It either creates the panel through `nav::add_panel` and focuses it, or, when a panel with the
same `PanelKey` is open, activates that tab and focuses it. Rows open by Enter and double-click
(`object_list/table.rs`, the Pods table), and links dispatch through the shared link path.

## Goals / Non-Goals

**Goals:**

- One open path with a `background` flag, so background and foreground opens can't disagree on
  placement or deduplication.

**Non-Goals:**

- Background open for the Resource panel's kind list. Opening a kind's list in the background is
  rarely useful. It can be added later with the same flag.
- An unread or new indicator on background tabs.

## Decisions

### 1. `OpenMode` on the single open path

`open_target_in` takes `mode: OpenMode { Foreground, Background }`, and every existing caller
passes `Foreground`.

- **Background, new panel:** call `nav::add_panel` with an option that inserts the tab without
  activating it, then skip `focus_panel`. If gpui-component's dock insert always activates the new
  tab, re-activate the previously active tab of that group in the same update, before the next
  render, so nothing flickers.
- **Background, already open:** return early. No activation, no focus.

Dedup, placement (`InsertTarget`), scope wiring (`watch_panel_focus`, `watch_scope_changes`) and
persistence stay shared.

- **Alternative: a separate `open_in_background` function.** Rejected, because it would be a third
  copy of the creation arm. The review of `panel-move-keybindings` already flagged the second.

### 2. Gestures

- **Rows.** The row `on_click` handlers read `event.modifiers().secondary()` (GPUI's
  platform-neutral `cmd`/`ctrl`) and `event.button == MouseButton::Middle`. A modified or middle
  click opens in the background and doesn't move the selection. A plain single click keeps its
  current behavior.
- **Keyboard.** `OpenInBackground` is registered per list context (`PodsPanel && !Input`,
  `ObjectListPanel && !Input`) with `secondary-enter`, so the keymap writes `cmd-enter` on macOS.
  The Pods hint row and the list hint rows show it.
- **Links.** The link click handler applies the same modifier and middle-click check, and calls
  `open_target_in(.., OpenMode::Background, ..)`.

## Risks / Trade-offs

- [gpui-component's insert may always activate the new tab] → the restore-previous-active step in
  decision 1, covered by a test asserting the list stays the active tab of its group.
- [`cmd-enter` may already be bound in a list context] → the prefix-aware conflict check runs in a
  test against every registered default.
