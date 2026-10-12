# Tasks

## 1. Keep the lines when the stream ends

- [ ] 1.1 Make `LogsView` tell a plain end from a failure: `LogEvent::Ended` records an ended
  state without taking over the panel, while `RequestFailed` and `NoPreviousInstance` keep their
  `terminal_message`. Verify: a `view` test applies lines then `Ended` and asserts the lines are
  still returned by `lines()` and the view reports ended; a `RequestFailed` test still reports its
  message and detail.
- [ ] 1.2 In `LogsPanel::render`, render `lines_list` for an ended view with a muted "log stream
  ended" row after the last line (or "The container wrote no log output." when there are no
  lines); keep `error_content` for failures only. Verify: a render test seeds fixed lines, applies
  `Ended`, and asserts the lines and the marker are present, in that order; a failed-request view
  still renders the error content and no lines.
- [ ] 1.3 Jump-to-bottom and follow's `scroll_to_item` count the marker row. Verify: a test
  applies `Ended` and asserts jump-to-bottom lands on the marker, not past it.

## 2. Non-follow for terminated containers

- [ ] 2.1 Add `follow: bool` to `LogTarget`; `log_params` sets
  `follow: target.follow && !target.previous`. Label-following panels pass `true`. Verify: a
  `log_params` unit test covers follow, non-follow, and previous.
- [ ] 2.2 `LogsPanel::restart` sets `follow: false` when the selected container's status is
  terminated (or the pod's phase is Succeeded/Failed), and `true` otherwise, including when no
  status is known. Verify: a test builds pods in each state and asserts the resulting target's
  `follow`.
- [ ] 2.3 `produce_container_logs` against a non-follow request still sends every line then
  `Ended`. Verify: the existing `stream` tests' mock-server harness gains a terminated-container
  case asserting all lines arrive before `Ended`.

## 3. Spec

- [ ] 3.1 Update the `pod-logs` delta so completed pods' output and the ended marker are
  specified. Verify: `openspec validate completed-pod-logs --strict` passes.

## 4. Full verification

- [ ] 4.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` all
  pass.
- [ ] 4.2 Manual smoke test: run a short-lived Job (for example one that echoes a few lines and
  exits), wait for the pod to show Completed, open its logs, and confirm the output is visible
  with "log stream ended" after it; repeat with a Job that writes nothing and one whose pod has
  been deleted (expect the not-found message).

## 5. Follow-up (not this change)

- Defaulting to the previous instance, or falling back to it, for a restarted pod whose current
  instance is terminated.
