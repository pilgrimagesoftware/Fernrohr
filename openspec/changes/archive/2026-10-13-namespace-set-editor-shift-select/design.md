# Design

## Context

`namespace-sets` shipped with the editor built on `ui::namespace_filter`'s `NamespaceFilter`
(`without_all` + `element_marked`), which already owns click handling for the list: `Command`'s
`on_confirm` reports a row's `IndexPath` and toggles that one row via `toggled()`. The same widget
backs the title-bar namespace picker, so anything added here benefits both.

See proposal.md for why; see the spec delta for the exact requirement text.

## Decisions

**Range state lives in `NamespaceFilter`, not the editor.** The anchor row (the last row clicked;
keyboard selection does not move the anchor - there is no shift-click equivalent for arrow+Enter)
joins the state `NamespaceFilter` already tracks (`selected`, the keyboard row), so the title-bar
picker gets range selection for free rather than needing its own copy.

**Reading shift requires an `on_mouse_down` ahead of `Command`'s own click handling.**
`Command`'s `on_confirm` reports an `IndexPath`, not a click's modifiers. gpui's base
`MouseDownEvent` always carries `Modifiers`, independent of gpui-kit's version, so a
`.on_mouse_down` on each row reads the shift key and records `pending_range: Option<usize>`
before `on_confirm` fires and applies it. Alternative considered: thread modifiers through
`on_confirm` itself - rejected without a confirmed `Command` API change; the mouse-down approach
needs no change to `gpui-kit` at all.

**A mixed-state range takes the anchor click's action, not each row's own state.** Shift-click is
"extend what I just did", matching Finder- and Gmail-style range selection: every row between the
anchor and the shift-clicked row ends up added if the anchor click added, removed if it removed -
not a per-row toggle, which would leave an unpredictable mix when the range already contains some
members and not others.

## Risks / Trade-offs

[Range selection only works by mouse, since there is no shift-click equivalent for keyboard
navigation] → Accepted: arrow+Enter already toggles one row at a time, which remains the
keyboard's full-coverage route; range select is a mouse-specific accelerator on top of it, not a
replacement.
