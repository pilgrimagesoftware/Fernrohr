# Tasks

## 1. Visible follow state

- [ ] 1.1 Restyle the `logs-follow` button in `stream_controls` (`util/logs/render.rs`) so on is a
  clearly filled/accented state and off is plain, using the app's existing accent for toggled
  controls. Verify: a render check confirms the button's toggled/styled state matches
  `FollowState::Following` and differs when `Paused`.
- [ ] 1.2 Make the tooltip state-aware ("Following new lines - click to stop" when on, "Follow new
  lines as they arrive" when off). Verify: a test asserts the tooltip text for each state.
- [ ] 1.3 Confirm the indicator flips with the existing pause/resume paths (wheel up pauses,
  returning to the end resumes). Verify: a test drives `on_lines_scrolled` and asserts the
  rendered state follows `follow_state()`.

## 2. Follow-on scrolls to the bottom

- [ ] 2.1 Add a `LogsPanel` helper (in `util/logs/panel.rs`) that scrolls to the newest visible
  line unless `scroll_handle.is_scrolled_to_end()` is already `Some(true)`; use it from the Follow
  click, `Jump to bottom`, and the resume path in `on_lines_scrolled`. Verify: unit tests cover
  already-at-bottom (no scroll) and not-at-bottom (scrolls to the last index).
- [ ] 2.2 In the `logs-follow` click handler, turning follow on calls the helper after
  `LogsView::scroll_to_bottom`; turning it off only calls `scroll_up`. Verify: a test seeds many
  lines, pauses follow, clicks Follow, and asserts the list is at the last line without any new
  line arriving.

## 3. Full verification

- [ ] 3.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` all
  pass.
- [ ] 3.2 Manual smoke test: stream a busy pod's logs, scroll up and confirm Follow shows off,
  click Follow and confirm it shows on and the view jumps to the newest line at once, scroll up
  again and confirm it flips off, click Follow off and confirm new lines no longer move the view.
