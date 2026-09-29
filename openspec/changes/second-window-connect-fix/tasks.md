# Tasks

## 1. Reproduce and isolate

- [x] 1.1 Write a failing test that opens two windows, connects the first to context `X`, then
  opens the second window's picker and selects the same already-connected context `X`; assert the
  second window transitions to `Workspace` mode.
  `second_window_connecting_to_an_already_connected_context_shows_workspace` in `shell.rs`'s test
  module. **Did not fail** - hypothesis one is not the cause. Drives the real
  `ClusterPicker::select` -> `PickerEvent::Connected` -> `watch_picker` -> `enter_workspace` path
  end to end (via `cx.add_window` + `watch_picker`, sharing one stub connection entity across two
  separately-constructed `MainWindow`s), which no existing test did - `connected_window`'s helper
  shortcuts straight to `enter_workspace`, bypassing this path entirely. This test is real, useful
  coverage regardless of the bug's status.
- [x] 1.2 Write a second test connecting the second window to a *different*, not-yet-connected
  context, and assert it transitions once that connection completes.
  `second_window_connecting_to_a_fresh_context_shows_workspace`. **Also did not fail** -
  hypothesis two is not the cause either. Drives a stub that starts `Connecting` and flips to
  `Connected` after construction (via `entity.update` + `cx.notify()`), exercising `cx.observe`'s
  callback rather than `select`'s synchronous `emit_connected` path.
- [x] 1.3 Neither test reproduces the reported symptom. The `select`/`watch_picker`/
  `enter_workspace` logic itself is correct for both the "already connected" and "connects later"
  cases, tested through the exact code path `NewWindow`'s `cx.on_action` handler uses
  (`open_window(cx, WindowLayout::default())` - same `open_window` as app startup).
  **Not resolved**: the bug, if still present, lives somewhere these tests can't reach - most
  likely something specific to two *real* OS windows via `cx.open_window` (vs. this harness's
  `cx.add_window`, which may not replicate every aspect of multi-window GPUI behavior), or the bug
  predates commits since folded into this lineage and may already be fixed. This environment has
  no attached WindowServer (confirmed in `cluster-picker-and-navigation/tasks.md` 6.2/15.2 - the
  same constraint blocking those manual smoke tests), so a real two-window manual test is required
  to confirm either way before claiming this fixed or still broken.

## 2. Fix

- [ ] 2.1 **Blocked on manual reproduction** (needs a real interactive desktop session - not
  available in this environment). If the symptom still reproduces manually, capture exactly which
  step differs from the two tests above (real second `cx.open_window` call, keyboard vs. menu
  trigger, timing) and write a third test that does reproduce it before touching any code.
- [ ] 2.2 Confirm any fix does not regress the already-passing single-window connect path, and
  keeps both tests from section 1 passing.

## 3. Full verification

- [x] 3.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` all
  pass (184/184, excluding the pre-existing unrelated
  `placeholder_remembers_the_kind_it_was_opened_for` flake - logged in `~/code/papercuts.md`),
  including both new regression tests.
- [ ] 3.2 Manual smoke test: launch the app, open a second window (Window menu or its
  keybinding), connect it to a context - including one already connected in the first window -
  and confirm it shows the workspace, not a stuck "connected" picker. **Needs a real interactive
  desktop session to complete.**
