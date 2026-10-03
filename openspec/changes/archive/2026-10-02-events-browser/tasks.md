# Tasks

## 1. Events data

- [x] 1.1 Add a typed shared Event watch (`WatchKey::Events`) per context via `watch_stream::run`, with rows carrying last seen, type, reason, involved object, message, count and source. Verify with a mock-server test covering both event APIs' fields, add/update/delete, and sharing between two subscribers.

## 2. Panel

- [x] 2.1 Add the events browser panel: live table sorted by age ascending (newest first) by default, re-sortable, with the sort saved, column set above, namespace scope, routing of the core Event kind to it, and an "Events" command. Verify with window-level tests for opening from the Resource panel and live updates.
- [x] 2.2 Involved object as a link opening its detail panel; a detail strip showing the selected event's full, selectable message. Verify with tests following a link and reading a long message.

- [x] 2.3 Add an Event section to object-detail (`sections/` dispatch): type badge, reason, full message, count, first/last seen, reporting component/instance, action, involved and related objects as references, falling back to `events.k8s.io/v1` fields via `events.rs::event_time` and friends. Verify with section tests for a legacy-field event and a v1-only event.

## 3. Filters and search

- [x] 3.1 Type, kind and reason multi-select filters built from present values, shown as active chips, clearable singly and together, saved with the panel. Verify with tests for warnings-only, combined filters, and restore.
- [x] 3.2 Search per design D3. Verify a message search narrows rows and shows a count.
- [x] 3.3 Keyboard routes and hint row for selection, link, search and each filter. Verify with a keystroke-only filtering test.

## 4. Verification

- [x] 4.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` pass.
- [x] 4.2 Manual check on a real cluster: open Events, filter to Warnings in one namespace, search a message, follow an event to its object.
  Passed 2026-10-02 (user).
