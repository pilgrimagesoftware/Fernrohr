# Design

## Context

`ui/panel/title.rs::namespace_picker` builds a `Button` with a `dropdown_menu` whose
`PopupMenuItem`s are rebuilt on every open from `namespaces_offered` (`None` for "All namespaces",
then the sorted names). Each item computes the next selection (`toggle`, or clear for "All")
and calls `on_pick(next, cx)`. `gpui_kit`'s `PopupMenu` has no filter input, so the menu can't
simply gain one.

`gpui-component` 0.6.6, which `gpui_kit` re-exports, ships `Combobox`/`ComboboxState` over a
`SearchableListDelegate`. It has a built-in search input, keyboard navigation, and a
`.multiple(true)` mode that doesn't close on selection.

## Goals / Non-Goals

**Goals:**
- Filtering, keyboard navigation and multi-select from one existing widget, with no hand-built
  popover.
- Keep `namespace_picker`'s external contract: the caller passes the scope, the namespaces and
  `on_pick(Vec<String>)`, and gets an element or `None` for cluster-scoped kinds.

**Non-Goals:**
- Sharing `resource-list-search`'s matcher. A namespace filter is a fixed case-insensitive
  substring match on one string, and coupling the two changes would order their delivery for no
  gain.

## Decisions

### Use `Combobox` in multi-select mode

*Alternatives:*
- A `Popover` hosting an `Input` plus a hand-rolled list. That means re-implementing keyboard
  navigation, focus and empty states that `Combobox` already has.
- Keeping `PopupMenu` and putting a filter input beside the button. The input would be detached
  from the list it filters, and take title-bar space.

Spike first (task 1.1). Confirm that the delegate can (a) pin "All namespaces" outside the
filter, (b) render a checked state per item, (c) customise the empty-state text, and (d) take an
Escape that clears the text before closing. If any of these isn't supported, fall back to the
`Popover` + `Input` + list alternative. The spec doesn't change either way.

### The selection lives in the panel, not the widget

`ComboboxState` keeps its own selection, but the panel's `PanelScope.namespaces` is the source of
truth (it's persisted and shared with the warp commands). On each change, translate the widget's
event into the same `next` vector the current code computes, call `on_pick`, and re-seed the
widget's selection from the scope on render. This keeps one owner and avoids drift when a warp
command changes the scope while the picker is closed.

### "All namespaces" is pinned, and clears the selection

The delegate's filtered items always prepend the `None` entry. Picking it calls `on_pick(vec![])`
as today. It's shown checked when the scope is empty.

### State lifetime

`ComboboxState` is an entity, unlike the per-render menu today, so the panel owns one per title
bar, created lazily. The filter text resets on open (spec: "Filter resets on reopen").

## Risks / Trade-offs

- [`Combobox`'s trigger looks different from the current ghost xsmall button] → Use its
  trigger-render hook (`ComboboxTriggerContext`) to keep the existing label, chevron and size.
- [A `Combobox` multi-select toggle closes or re-sorts items] → Covered by the spike. The fallback
  is the popover.
- [Very large namespace lists] → The searchable list is virtualized, and the filter is a substring
  check per name.
