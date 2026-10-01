# Tasks

## 1. Matcher

- [ ] 1.1 Add `regex` as a direct dependency, and a GPUI-free `SearchQuery` (text plus mode,
  scope, case_sensitive, regex) compiled to a substring, a `Regex` or `Invalid`, with a
  `Searchable` trait (name, visible values for given columns, labels). Verify: unit tests cover
  the default case-insensitive substring, case sensitive, regex, an invalid regex matching as if
  the box were empty, and each of the three scopes, including Name not matching a Status-only hit
  and Labels matching `key=value`.

## 2. Search box component

- [ ] 2.1 Add a `ListSearch` entity rendering the header input, the options menu (mode, scope, two
  toggles) and the `matched / total` count, with an error style for an invalid regex. Escape
  clears the box and returns focus to its owner. Verify: a GPUI test types into it, toggles each
  menu option, and asserts that the resulting `SearchQuery` and the count text update. Another
  test asserts that Escape empties the box.
- [ ] 2.2 Persist the menu defaults as `UiConfig.list_search` with `#[serde(default)]`. A new
  `ListSearch` reads them, and a menu change saves them without touching open panels. Verify: a
  test round-trips the settings through `ui.toml`, an existing `ui.toml` without the key still
  loads, and a second open `ListSearch` keeps its own settings after the first one changes.

## 3. Pods panel integration

- [ ] 3.1 Put `ListSearch` in the Pods panel header, and register `pods.focus_search` bound to `/`
  in the Pods key context. `PodTableRow` implements `Searchable` using the table's visible
  columns. Filter mode drops non-matching rows before `sync_table`. Remove the unwired
  `view_rows`/`matches_filter`. The text clears on a kind change and survives a namespace change.
  Verify: tests assert that `/` focuses the box, that `nginx` filters the rows and the count, that
  a namespace switch keeps the text, and that a kind switch clears it.
- [ ] 3.2 Highlight mode: every row stays visible, matched rows get the accent tint, and Enter or
  Shift+Enter in the box moves the table selection to the next or previous match, wrapping around
  in sorted order. Verify: a test with three matches asserts which rows are tinted, the selection
  after Enter, the wrap back to the first match, and the selection after Shift+Enter.
- [ ] 3.3 Update `docs/` or the in-app shortcuts list if they enumerate panel commands, so the new
  `/` binding shows up. Verify: the keybindings editor lists `pods.focus_search`.

## 4. Full verification

- [ ] 4.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` all
  pass.
- [ ] 4.2 Manual smoke test on a cluster with many Pods: filter by name, by a Status value under
  Visible columns, and by a label. Switch to Highlight and step through the matches. Enter an
  invalid regex. Restart, and confirm the menu settings came back. Check the tint against the
  selection colour in both light and dark themes.
