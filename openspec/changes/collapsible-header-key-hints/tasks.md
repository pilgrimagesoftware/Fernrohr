# Tasks

## 1. The collapsible hint row

- [x] 1.1 Add `consts::MAX_HINT_LINES = 3` and a `HintRow` component (D1, D2):
  - hints added as essential or not;
  - wrapped line count measured in prepaint;
  - collapsed state with D1's hysteresis;
  - the essential hints (or the first, when none is marked) shown when collapsed.

  Verify with unit tests over the packing function: three lines fits, four collapses, and a
  collapsed row expands only when the measured widths fit in three. Add a UI test that a narrow
  row draws only its essential hints.
- [x] 1.2 Add the overflow control (D3, D4): an icon button, last in the row, a tab stop with a
  tooltip listing the hidden hints' live keys and naming `?`. Enter or Space dispatches
  `ShowKeyHints`. Verify with tests:
  - the tooltip lists the hidden hints and their keys;
  - with real keystrokes (`simulate_keystrokes`), Tab reaches the control and Enter opens the Key
    Hints overlay;
  - a hidden hint's key still runs its command.

## 2. Use it in every wrapping hint row

- [x] 2.1 Build the pod detail and object detail headers' hints with `HintRow`, keeping
  `header_title`'s layout. Essential: the view switches and the view toggle. Verify with a narrow
  detail panel test: collapsed to those hints plus the control, header height within three lines.
- [x] 2.2 Build the Pods, object list and events browser hint rows with `HintRow`. Essential: Open
  (where shown), Filter and Describe. The object list's row now wraps. Verify with narrow-panel
  tests for each. Also check that a hidden Port-forward hint is listed in the Pods row's overflow
  tooltip (D5).
- [x] 2.3 Check that the shell panel's hint row, dialogs, the Logs control bar and the picker key
  row are unchanged. Verify with a narrow shell panel test: every reserved key still shows.

## 3. Documentation

- [x] 3.1 Document it:
  - in `docs/site/latest/details.md` and `docs/site/latest/resources.md`, how a narrow panel's
    hints collapse to the essential ones, the overflow control (hover for the hidden keys, Tab and
    Enter for the Key Hints overlay), and that every key still works;
  - in `docs/site/latest/reference/keybindings.md`, that `?` lists hidden hints too.

  Use placeholders only. Verify: the docs-site check (`bundle exec ruby _scripts/check.rb`) and
  `cargo test` pass.

## 4. Integration

- [x] 4.1 Run `cargo fmt -- --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test`
  in `App/`. Verify: all pass.
- [ ] 4.2 Check manually at the default and the largest text size: narrow a pod detail panel and a
  Pods list until their hints collapse. Hover and Tab to the control, and check every hidden key
  still works. Then widen the panels and check they expand without flicker. Record no
  cluster-identifying details.
