# Proposal

## Why

The Pods table renders every cell in the monospace font (`PodTableDelegate::render_td` sets
`cx.theme().mono_font_family` unconditionally), which is wrong for a data table of short
structured values - monospace is for YAML/logs, not a name/status/age grid. Separately, the table
is explicitly built with `.sortable(false).col_movable(false).col_resizable(false)`, so columns
can't be resized, reordered, or clicked to sort - basic table affordances every other resource
browser (FreeLens, k9s, Lens) provides, and which gpui-component's `TableState`/`Column` already
support end to end (including a built-in sort-direction indicator icon) - this is flipping
existing flags and implementing one delegate hook, not building new table infrastructure.

## What Changes

- Pods table cells render in the UI font (drop the explicit mono `font_family` override), plain
  values styled like any other data grid.
- Columns become resizable and reorderable (`col_resizable(true)`, `col_movable(true)`).
- Columns become sortable by clicking their header, ascending/descending, with the sort key shown
  via gpui-component's built-in indicator icon (`ColumnSort`/`render_sort_icon`) - implemented via
  `PodTableDelegate::perform_sort`, reusing the existing `sort_rows`/`SortState` machinery already
  written for `view_rows` (currently unwired dead code per its own doc comment) rather than adding
  a second sort implementation.

## Capabilities

### Modified Capabilities
- `resource-browser`: adds the requirement that a resource table's columns are resizable,
  reorderable, and sortable by clicking a header, with a visible indicator of the current sort
  key and direction.

## Impact

- `app/src/k8s/resource/pods.rs`: `PodTableDelegate::render_td` (font), `TableState::new(...)`
  builder call (the three `false` flags), and a new `perform_sort` impl wiring to the existing
  `sort_rows`/`SortState` helpers.
