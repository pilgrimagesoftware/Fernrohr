# Proposal

## Why

The namespace picker in a panel's title bar is a plain dropdown menu with one checkable item per
namespace. On a cluster with dozens or hundreds of namespaces, finding the right one means
scrolling a long menu by eye. Every other long list in the app is filterable or headed that way:
the Resource panel's kind filter, the command palette, and `resource-list-search` for list panels.

## What Changes

- The namespace picker opens with a filter input at the top. Typing narrows the namespace list to
  names containing the text, ignoring case.
- "All namespaces" stays pinned at the top of the list whatever the filter says, so clearing the
  scope is always one click away.
- The picker stays multi-select. Toggling a namespace leaves the picker open, and the filter text,
  so several matches can be checked in a row. The button label is unchanged ("All namespaces", one
  name, or "N namespaces").
- Keyboard: the input has focus when the picker opens. Up and Down move through the filtered
  list, Enter toggles the highlighted namespace, Escape clears the filter text, and a second
  Escape closes the picker.
- A filter matching nothing says so, rather than showing an empty list.
- Out of scope: regex or scope options (namespaces are just names), a "select all matching"
  action, and the include/exclude namespace sets of `namespace-include-exclude`. That change
  alters what a pick means, and this one only changes how a namespace is found.

## Capabilities

### New Capabilities

### Modified Capabilities
- `resource-browser`: adds a requirement that the namespace picker can be filtered by typing.

## Impact

- `app/src/ui/panel/title.rs`: `namespace_picker` swaps its `Button` + `dropdown_menu` for
  `gpui_kit`'s `Combobox` in multi-select mode, with a `SearchableListDelegate` over
  `namespaces_offered`. `label_for` and the `on_pick` contract stay as they are.
- The `ui/panel/title/tests.rs` picker tests are updated for the new widget.
- No persistence or config changes.
