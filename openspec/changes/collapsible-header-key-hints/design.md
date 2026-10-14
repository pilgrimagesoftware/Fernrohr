# Design

## Context

See proposal.md - Why. The current code that shapes the approach:

- App#282 made the pod detail and object detail headers share `panel_title::header_title` and
  `header_hints` (`ui/panel/title/header.rs`). The title takes the width it needs first, and the
  hints fill the rest as a `flex_wrap` row with a 14rem floor. Nothing caps how many lines the
  hints wrap onto.
- The list hint rows are built per panel:
  - the Pods panel's `pods/hints.rs` (wraps);
  - the object list's `object_list/render.rs` (doesn't wrap);
  - the events browser's `render.rs` (wraps).

  Each hint is a `Kbd` read from the live keymap (`Kbd::binding_for_action`) beside a label.
- Every hint is a registered command's key, per the keyboard-first rule. `?` (`global.show_key_hints`,
  `util/key_hints.rs`) already opens a read-only overlay listing every command available where
  focus is, with its key. The command palette lists the same commands and runs them.
- Rows that aren't wrapping hint rows: dialog buttons with their `Kbd`, the Logs panel's control
  bar (a key beside each control), the picker's key row, and the shell panel's hint row, which
  `exec-terminal` requires to list its reserved keys.

## Goals / Non-Goals

**Goals:**
- One collapsible hint row component that every wrapping hint row uses, so the threshold,
  overflow control and keyboard route are decided once.
- No hint becomes unreachable: collapsing hides a label, never a command or its key.

**Non-Goals:**
- Changing which commands a row lists, or any key.
- Collapsing keys shown beside individual controls (dialogs, Logs, the picker), or the shell
  panel's row.
- A per-user setting for which hints are essential.

## Decisions

### D1: Measure the wrapped lines; collapse past `MAX_HINT_LINES`

A new `consts::MAX_HINT_LINES: usize = 3` sets the limit. During prepaint, the row records each
hint's bounds and counts the distinct row offsets they land on. That count is the row's line
count, measured with the actual font, text size and width, never estimated from a hint count. If
it exceeds the limit, the row renders collapsed from the next frame.

To avoid flicker at the boundary, a collapsed row expands again only when the full set would fit.
It keeps every hint's last measured width, and expands once those widths pack into
`MAX_HINT_LINES` lines at its current width, using the same gap as the row. It re-measures after
expanding.

*Alternative considered:* collapse when the panel is narrower than a fixed width. Rejected: the
right width depends on text size, labels and which hints show (Shell only with a running
container, for example), so a fixed width would be wrong at every text size but one.

### D2: Each row marks its essential hints, not a flag on the command

Each hint row's builder marks hints essential as it adds them (for example,
`HintRow::essential(hint)` beside `HintRow::hint(hint)`), and the collapsed row keeps them in row
order.

*Alternative considered:* an `essential: bool` on the registered `Command`. Rejected:
- Whether a hint is essential depends on the panel, not the command. Describe (`d`) is central in
  a list and absent from a detail header, and a detail header's view toggle has no list
  counterpart.
- Some hints aren't one command's key: the pod detail header's tab keys are shown as one group.

The row already owns its hints' order and conditions, so it owns which matter most too. If a row
marks nothing essential, its first hint stays, so a collapsed row is never empty.

Initial marks: detail headers keep the view switches (tab keys) and the YAML/structured toggle;
the Pods, object list and events browser rows keep Open (where shown), Filter and Describe.

### D3: The overflow control is an icon button

The overflow control is an icon button with the ellipsis icon the dock's own toolbar menu uses.
It is sized like a hint and placed last in the row. Its tooltip, per the icon-buttons rule, reads
"More keys". It then lists each hidden hint as its live `Kbd` and label, in row order. The
tooltip names `?` from the live keymap ("Press ? for every key"), so it also teaches the keyboard
route.

### D4: Keyboard: a tab stop that opens Key Hints

The control is a tab stop inside its panel's focus trap. Enter or Space on it dispatches
`ShowKeyHints`, which opens the existing overlay for the panel the control is in. That overlay
already lists every available command, hidden hints included, with keys, and dismisses with
Escape back to the control. No new command is added: `?` is already registered, in the palette
and rebindable, and it reaches the hidden hints from anywhere in the panel, with or without the
control.

Hidden hints remain registered commands: their keys work while hidden, and the command palette
still lists and runs them.

*Alternative considered:* a popover listing just the hidden hints, opened by Enter. Rejected:
the `?` overlay already does this for every key, and a second list of the same keys would drift.

### D5: Hidden still counts as in the hint row

Several specs say a command "appears in the hint row": `port-forward-indicators`,
`resource-browser` and `events-browser`. `key-hint-rows` states that a collapsed row's hidden
hints still count as listed, through the overflow control. Those requirements don't need
modifying, and a narrow panel doesn't break them.

## Risks / Trade-offs

- [The row's height changes between frames as it collapses] → D1 decides from the previous
  frame's measurement, with hysteresis, so it settles in one frame and doesn't oscillate at the
  threshold.
- [Hiding a hint the user relies on] → Its key keeps working, the tooltip names it, and `?` lists
  it. The essential marks (D2) start conservative and are per row, easy to adjust.
- [The object list's row starts wrapping] → It's needed for the collapse to apply; an object list
  with few hints looks the same as before.
