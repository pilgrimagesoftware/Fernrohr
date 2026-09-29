# Tasks

## 1. Font

- [x] 1.1 Remove `PodTableDelegate::render_td`'s explicit `.font_family(cx.theme().mono_font_family
  .clone())` cell override. Verify: a render-level check (or visual smoke test) shows table cells
  in the UI font, not monospace.

## 2. Sorting

- [x] 2.1 Confirm gpui-component's `col_ix` passed to a delegate is a stable per-column identity
  (matching `Column`'s own `id`), not a post-reorder visual position - resolve `render_td`'s
  positional match arms to that identity if it isn't already. Verify: a test (or render check)
  reorders two columns and asserts each still shows its own data - it is **not** an identity:
  gpui-component 0.6.6's `TableState::move_column` reorders its own `col_groups` and calls
  `TableDelegate::move_column(col_ix, to_ix)`, expecting the delegate to reorder its own column list,
  and every later `column`/`render_td`/`perform_sort` call passes the visual position. New
  `k8s/resource/pods_table.rs` (split out of `pods.rs`): a closed `PodColumn` enum, a delegate that
  keeps `columns: Vec<PodColumn>` and resolves every `col_ix` through it, and `move_column` reordering
  it. Test `moving_columns_renders_each_visual_position_from_its_own_column` drives the same lookup
  `render_td` uses. Found alongside: `sync_table` called `TableState::refresh()` on every row update,
  which rebuilds columns from `delegate.column()` and would have reset widths, order and the sort
  indicator on each watch delta; `rows_count` is read live, so the call is dropped
- [x] 2.2 Implement `PodTableDelegate::perform_sort`, mapping the clicked column to a `SortState`
  field and cycling `Default -> Ascending -> Descending -> Default`, then calling the existing
  `sort_rows` helper before rows reach the table. Verify: a test clicks each sortable column in
  turn and asserts row order matches `sort_rows`'s own behavior for that `SortState` - the cycle is
  gpui-component's, not ours: `TableState::perform_sort` goes Default -> Descending -> Ascending ->
  Default and hands the delegate the new direction, so the delegate applies what it's given and
  the header arrow always matches the rows. `sort_rows` and the delegate share one pure `compare`
  (gaining the IP and Node columns it lacked); descending uses a reversed comparator so ties keep
  their order; Default restores the rows' incoming order. Tests
  `compare_orders_rows_by_every_column`, `resort_orders_rows_and_default_restores_the_supplied_order`,
  `set_rows_keeps_an_active_sort_applied`, `a_row_update_keeps_a_real_table_states_rows_sorted`
- [x] 2.3 Enable `.sortable(true)` on the table and mark each `Column` sortable via gpui-component's
  own `Column::sortable()`/`sort()` builder. Verify: a render check confirms the sort indicator
  icon appears on the active sort column and reflects direction - every column is `sortable()`, and
  `column()` re-applies the active direction so any future `refresh()` keeps the indicator. Test
  `column_reports_the_active_sort_on_the_active_column_only`

## 3. Resize and reorder

- [ ] 3.1 Enable `.col_resizable(true)` and `.col_movable(true)`. Verify: a manual/visual check
  drags a column boundary and a column header and confirms both behaviors work.

## 4. Full verification

- [ ] 4.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` all
  pass.
- [ ] 4.2 Manual smoke test: open a Pods table, confirm UI-font cells, resize a column, reorder
  two columns and confirm data follows the column not the position, click a header to sort
  ascending then descending and confirm the indicator updates.
