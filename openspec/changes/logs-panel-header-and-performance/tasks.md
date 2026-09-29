# Tasks

## 1. Title

- [ ] 1.1 `LogsPanel::title`/`tab_name` build `"Logs: <pod> · <container>"` from `self.current`
  when it is `Some`, falling back to the existing generic label otherwise. Verify: a test sets
  `current` to a fixture pod/container and asserts the title includes both; a test with no
  `current` set asserts the existing fallback text.

## 2. Virtualized scrolling

- [ ] 2.1 Replace `LogsPanel::render`'s unconditional `view.lines().iter().cloned().map(...)`
  `.children()` block with `gpui::uniform_list` (or gpui-component's equivalent, whichever this
  app already depends on), rendering only the visible row range. Preserve the existing
  `.whitespace_nowrap()` monospace one-line-per-row styling and `.overflow_scrollbar()`-equivalent
  scroll behavior. Verify: a render test with a large synthetic line count asserts the visible
  element count stays bounded rather than matching total line count.
- [ ] 2.2 Manual check: stream (or seed) a log panel with several thousand lines and confirm
  scrolling stays responsive.

## 3. Full verification

- [ ] 3.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` all
  pass.
- [ ] 3.2 Manual smoke test: open a Pod's logs, confirm the title shows pod and container, confirm
  scrolling a long log stream feels smooth.
