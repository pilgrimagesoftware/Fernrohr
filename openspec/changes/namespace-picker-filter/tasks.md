# Tasks

## 1. Widget spike

- [x] 1.1 Confirm `gpui_kit`'s `Combobox` multi-select can pin an unfiltered first entry, show
  per-item checked state, customise the empty message, and clear the text on the first Escape
  before closing on the second, all through a `SearchableListDelegate` and its options. Record the
  outcome, `Combobox` or the `Popover` + `Input` fallback, in `design.md`. Verify: a scratch GPUI
  test (deleted or kept as the basis of 2.x) exercises each of the four points.

## 2. Filterable picker

- [x] 2.1 Rebuild `namespace_picker` on the chosen widget, keeping its signature, `label_for` and
  the pinned "All namespaces" entry. The panel owns the picker state, which re-seeds its selection
  from `PanelScope.namespaces`, and toggles still call `on_pick`. Verify: the existing
  `ui/panel/title/tests.rs` picker tests pass, updated for the new widget, and a new test asserts
  that typing `KUBE` lists only "All namespaces" and the `kube-*` names.
- [x] 2.2 Multi-pick and keyboard: toggling keeps the picker open with its filter, Up/Down/Enter
  toggle the highlighted entry, Escape clears and then closes, a no-match filter shows "No
  matching namespaces", and the filter resets on reopen. Verify: one test per scenario in
  `specs/resource-browser/spec.md`'s "Filtering the namespace picker".
- [x] 2.3 The button's look is unchanged: same label, chevron, size and ghost style. A warp command
  that changes the scope while the picker is closed is reflected the next time it opens. Verify: a
  test runs a warp, then opens the picker and asserts the checked entries.

## 3. Full verification

- [x] 3.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` all
  pass.
- [ ] 3.2 Manual smoke test on a cluster with many namespaces: filter, pick two matches without
  the picker closing, use the keyboard only, check the empty state, and confirm the title-bar
  button looks as before.
