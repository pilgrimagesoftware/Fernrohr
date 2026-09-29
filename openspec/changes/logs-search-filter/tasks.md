# Tasks

## 1. Filter input

- [ ] 1.1 Add an `Entity<InputState>` to `LogsPanel`, rendered via `gpui_component::input::Input`
  in the control bar alongside the container picker/follow/jump controls. Verify: a render check
  confirms the input is present and focusable.
- [ ] 1.2 Filter `view.lines()` by the input's current text (case-insensitive substring) before
  computing `uniform_list`'s `item_count`; index the row-render closure into the filtered set, not
  the raw one. Verify: a test seeds fixed lines, sets a filter, and asserts only matching lines
  are counted/rendered.

## 2. Interaction with follow/jump

- [ ] 2.1 Jump-to-bottom and the Follow toggle's auto-scroll use the *filtered* count while a
  filter is active. Verify: a test applies a filter that excludes the newest line and asserts
  jump-to-bottom lands on the last *matching* line, not past the filtered list's end.

## 3. Full verification

- [ ] 3.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` all
  pass.
- [ ] 3.2 Manual smoke test: stream a pod's logs, type a filter, confirm only matching lines show,
  clear it, confirm full history returns, confirm follow/jump behave correctly while filtered.

## 4. Follow-up (not this change)

- Regex filtering.
- Highlighting the matched substring within each visible line.
