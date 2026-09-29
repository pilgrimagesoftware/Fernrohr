# Design

## Font: delete the override, don't add a new one

`render_td` sets `.font_family(cx.theme().mono_font_family.clone())` on every cell explicitly.
There is no counterpart "UI font" override needed - removing this line lets the cell inherit
whatever font the table/panel ambient context already uses (the app's UI font, once
`app-menu-and-fonts` wires it; today's platform default in the meantime). This is a deletion, not
new plumbing.

## Sort: reuse `sort_rows`/`SortState`, don't invent a second sort path

`sort_rows`/`SortState`/`view_rows` already exist in this file, written for a filter/sort surface
this panel doesn't render yet (`view_rows` carries an `UNWIRED` doc comment saying exactly that -
"`PodsPanel::render` doesn't call this yet"). `perform_sort(col_ix, ...)` - the delegate hook
gpui-component's `TableState` calls when a header is clicked - becomes the *first* real caller:
map `col_ix` to the matching `SortState` field/column, toggle direction the same way
`state::perform_sort`'s own `ColumnSort::Default -> Ascending -> Descending -> Default` cycle
does (matching library convention rather than inventing a different cycle), store the resulting
`SortState` on `PodTableDelegate`, and have the existing per-render row-building step call
`sort_rows` with it before handing rows to the table. No second sort implementation, no
duplicated comparison logic between an unwired one and a new one.

## Resize/reorder: flip the flags, verify the delegate doesn't assume fixed columns

`col_resizable(true)`/`col_movable(true)` are gpui-component's own builder flags - the library
handles drag-resize and drag-reorder interaction entirely. The one thing to verify on this app's
side: `render_td`'s `col_ix` match arm assumes columns stay in registration order (`0 => name, 1
=> namespace, ...`). If the library reports `col_ix` as the *current visual position* after a
reorder rather than a stable per-column identity, this hardcoded match silently shows the wrong
value in a reordered column - worth confirming against gpui-component's actual behavior (its
`Column` type carries an `id`, which is what should ground this lookup instead of positional
`col_ix`, if that's what the library expects for delegates supporting reordering).
