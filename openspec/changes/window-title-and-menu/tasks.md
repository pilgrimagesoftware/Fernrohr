# Tasks

All work is in the `App` submodule (`Fernrohr-App`), on the branch this change's section numbers
name. `cargo test` and `cargo clippy -- -D warnings` are the gates.

## 1. The title function

- [ ] 1.1 Add `util/shell/window_title.rs` with a pure `title_for(&WindowMode) -> String` returning
      the app name alone in `Picker` mode, `"<context> - <app>"` for a single context, and
      `"<n> clusters - <app>"` for several (design.md decision 4). Take the app name from the same
      source `ui/menu.rs`'s About view uses, so the two cannot disagree on casing.
- [ ] 1.2 Unit-test `title_for` in `util/shell/window_title/tests.rs` for all three modes and for
      the multi-context case naming the count rather than a context. Pin the exact expected strings,
      including the app name's casing (design.md Risks, last bullet). Verify with `cargo test window_title`.
- [ ] 1.3 Declare the new module in `util/shell.rs` so it compiles in the shell's module tree, and
      verify `cargo build` is clean before moving on.

## 2. Titling at open

- [ ] 2.1 In `util/shell/window.rs`, replace `open_window`'s `..Default::default()` with
      `titlebar: Some(TitlebarOptions { title: Some(title_for(&mode).into()), appears_transparent:
      false, traffic_light_position: None })` (design.md decision 2), building the initial mode
      before the window opens so a restored multi-context window is titled correctly on its first
      frame rather than blank.
- [ ] 2.2 Verify by inspection that no other `WindowOptions` in `util/shell/window.rs` still relies
      on `titlebar: None`, and that the workspace and picker paths both reach the same options.

## 3. Titling on change

- [ ] 3.1 Add a helper in `util/shell/window_title.rs` that pushes a window's current title through
      `window.window_handle().update(cx, |_, window, _| window.set_window_title(&title))`, taking
      the title from the window's own mode so no caller can pass a stale one.
- [ ] 3.2 Call it from `MainWindow::sync_context_children`, inside the existing `cx.defer` and after
      the `Picker`/`contexts[active]` early returns (design.md decision 5), so `enter_workspace`,
      `add_context`, `disconnect_context` and `set_active_context` are all covered by one hook.
- [ ] 3.3 Call it from `MainWindow::enter_picker`, which sets `Picker` mode without syncing children,
      so a window that loses its last context drops back to the plain app-name title.

## 4. Tests

- [ ] 4.1 In `util/shell/contexts/tests.rs`, extend the existing `TestAppContext` cases to assert
      `window.window_title()` after each transition: entering a workspace names the context; adding
      a second context switches to the count; disconnecting one back to a single names the survivor;
      disconnecting the last returns the app name. Verify with `cargo test contexts`.
- [ ] 4.2 Add a case covering a window restored from a multi-context layout, asserting it is titled
      with the count from its first frame (the `app-shell` spec's "Restored windows come back
      titled"). Verify with the same command.
- [ ] 4.3 Run the full gate: `cargo fmt && cargo clippy -- -D warnings && cargo test`.

## 5. Proposal bookkeeping

- [ ] 5.1 Tick off the completed tasks in this repo's
      `openspec/changes/window-title-and-menu/tasks.md` as each section lands, per the meta repo's
      two-repo workflow.
