# Tasks

## 1. Live events

- [x] 1.1 Add a field-selected event watch for one involved object in `events.rs`, built on `watch_stream::run`, with a 403 surfacing as the existing "could not be listed" state. Verify with a mock-server test: initial events, an added event, a deleted event, and a 403.
  App#107: `events::watch` through `watch_stream::run_with`, into the events browser's `EventsTable`.
  Verified by `k8s::resource::events::watch_tests` (list, an added and a deleted event, the field
  selector required; a 403 as the refusal).
- [x] 1.2 Drive the pod detail Events tab from the watch, started on open and dropped on close. Verify with a window-level test that a new event appears without reopening, and that closing the panel stops the watch.
  `pod_detail::live_events`, started once the pod loads and dropped with the panel. Verified by
  `pod_detail::tests::live_events` (a new event shows with no reopen or refetch; closing the panel
  releases its table).

## 2. Time window

- [x] 2.1 Add the window (15m, 1h, 6h, 24h, All) as a client-side filter on `event_time`, re-evaluated each minute, with a "N older events hidden" line. Verify with unit tests on the filter and hidden count, and a test that an event ages out.
  `within_window` over `EventRow::last_seen`, undated events kept; a 60s tick re-renders. Verified by
  `pod_detail::tests::events_window`.
- [x] 2.2 Add the selector to the Events tab plus panel-scoped keys and palette commands, persisting `pod_events_window` in `ui.toml` (default 1h). Verify the selector by keyboard and that the preference round-trips across a relaunch.
  Selector row, `[`/`]` and one palette command per window, all panel-scoped; saved via
  `pod_detail::window_preference`. Verified by `pod_detail::tests::events_window_keys` (keys, a
  click, `ui.toml`, a relaunch).

## 3. Overview warnings

- [x] 3.1 Show the most recent Warning events within the window on the Overview tab, with a link to the Events tab, and nothing when there are none. Verify with window-level tests for the crashing-pod and quiet-pod cases.
  Up to three in-window warnings, newest first, with an "All events" link. Verified by
  `pod_detail::tests::overview_warnings` (crashing and quiet pods).

## 4. Verification

- [x] 4.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` pass.
  869 passed, 2 ignored.
- [x] 4.2 Manual check on a real cluster: delete a pod's container process or roll a deployment, watch events arrive live, change the window, restart and confirm it stuck.
  Passed 2026-10-02 (user).
