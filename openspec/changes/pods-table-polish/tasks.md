# Tasks

## 1. Font

- [x] 1.1 Remove `PodTableDelegate::render_td`'s explicit `.font_family(cx.theme().mono_font_family
  .clone())` cell override. Verify: a render-level check (or visual smoke test) shows table cells
  in the UI font, not monospace.

## 2. Sorting

- [ ] 2.1 Confirm gpui-component's `col_ix` passed to a delegate is a stable per-column identity
  (matching `Column`'s own `id`), not a post-reorder visual position - resolve `render_td`'s
  positional match arms to that identity if it isn't already. Verify: a test (or render check)
  reorders two columns and asserts each still shows its own data.
- [ ] 2.2 Implement `PodTableDelegate::perform_sort`, mapping the clicked column to a `SortState`
  field and cycling `Default -> Ascending -> Descending -> Default`, then calling the existing
  `sort_rows` helper before rows reach the table. Verify: a test clicks each sortable column in
  turn and asserts row order matches `sort_rows`'s own behavior for that `SortState`.
- [ ] 2.3 Enable `.sortable(true)` on the table and mark each `Column` sortable via gpui-component's
  own `Column::sortable()`/`sort()` builder. Verify: a render check confirms the sort indicator
  icon appears on the active sort column and reflects direction.

## 3. Resize and reorder

- [ ] 3.1 Enable `.col_resizable(true)` and `.col_movable(true)`. Verify: a manual/visual check
  drags a column boundary and a column header and confirms both behaviors work.

## 4. Full verification

- [ ] 4.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` all
  pass.
- [ ] 4.2 Manual smoke test: open a Pods table, confirm UI-font cells, resize a column, reorder
  two columns and confirm data follows the column not the position, click a header to sort
  ascending then descending and confirm the indicator updates.
