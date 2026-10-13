# Tasks

## 1. Pin down existing behaviour

- [x] 1.1 Add tests (or confirm existing `labels/tests.rs` ones) for: opening empty, invalid
  selector keeps the last one, select-all selector rejected, stream started for a pod that appears
  after the selector is applied, stream dropped when a pod disappears, cap of 20 with the
  "streaming 20" note. Verify: each spec scenario in `selector-logs` has a passing test.

## 2. Menu entry

- [x] 2.1 Add a menu item for `ShowLabelLogs` ("Open Logs by Label...") next to the other Logs
  entries; keep the palette command. Verify: a test shows the item in the menu model and invoking
  it opens the empty typed panel.

## 3. Pod colour

- [x] 3.1 Keep each line's source tag separate from its text in `LogsView`. Verify: a view test
  asserts the tag and text are stored apart and `lines()` consumers (follow, jump) still work.
- [x] 3.2 Add a pure `pod_colour(name) -> palette index` and draw the tag in that colour. Verify: a
  unit test asserts the same name always maps to the same index and containers of one pod match.

## 4. Container filter

- [x] 4.1 Add the filter to `Following` and `wanted`; render the picker in the bar. Verify: a test
  with a two-container pod asserts "All" streams both and a named container streams only it, with
  the stopped stream dropped.
- [x] 4.2 Save and restore the filter in the panel's layout state. Verify: a restore test round-trips
  it.

## 5. Namespace scope

- [x] 5.1 Surface the panel's namespace scope in the bar with a picker over the current
  namespace(s), namespace sets, and all; re-run `sync_labels` on change. Verify: a test switches
  scope and asserts streams for pods outside it stop.

## 6. Full verification

- [x] 6.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` all
  pass. Verify: all three exit zero.
- [x] 6.2 Manual smoke test: open the panel empty from the menu, type `app=x`, confirm matching
  pods stream with coloured prefixes, scale a deployment up and confirm the new pod streams and a
  deleted one stops, pick one container, change namespace scope, scroll up to pause follow, and
  restart the app to confirm the panel restores. Verify: each step behaves as described.

## 7. Follow-up (not this change)

- Regex pod-name filter.
- Exclude filters.
- `since` and tail options, configurable stream cap.
- Timestamp-ordered merge.
