# Tasks

## 1. Range selection

- [ ] 1.1 In `ui::namespace_filter::NamespaceFilter`, add an anchor-row field and an
  `on_mouse_down` on each row (ahead of `Command`'s own click handling) that records the clicked
  row as the new anchor on a plain click, or reads a pending range when shift is held; verify
  with a unit test that a plain click updates the anchor and a shift-click does not.
- [ ] 1.2 On a shift-click, apply the anchor row's last action (add or remove) to every row
  between anchor and target, inclusive, including the anchor row's own set membership as the
  source of truth for which action to apply; verify with tests covering an all-add range, an
  all-remove range, and a mixed-state range asserting it follows the anchor's action rather than
  each row's own prior state.
- [ ] 1.3 A shift-click with no anchor set yet (first interaction in a fresh editor session)
  ranges from the first listed row; verify with a test.
- [ ] 1.4 Run `cargo fmt && cargo clippy -- -D warnings && cargo test` and fix any failures.
