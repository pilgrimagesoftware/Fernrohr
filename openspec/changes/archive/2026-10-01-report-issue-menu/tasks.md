# Tasks

## 1. Investigation and Setup

- [x] 1.1 Examine existing command handler patterns in `app/src/ui/menu.rs` and elsewhere to determine if a `commands/` module exists; verify existing dependency list to confirm whether `open` crate is available
  - No `commands/` module; commands live beside their feature (`ui/link/go_to.rs` is the reference) and register via `CommandRegistry`/`actions!`, not inline in `menu.rs`. New file: `app/src/ui/report_issue.rs`. No `open` crate dependency or need for one - see 2.2.
- [x] 1.2 Verify that `build_info` module from `about-window` is accessible and exports the `FERNROHR_BUILD_COMMIT` and `FERNROHR_BUILD_DATE` values; confirm they're available at runtime
  - `version()`/`build_identifier()` widened from `pub(super)` to `pub(crate)` and re-exported from `about_window::mod` so `report_issue` can reuse them directly - no duplicated build-time logic.
- [x] 1.3 Check existing patterns for OS version detection (e.g., does the app already use `sys-info`, `uname`, or std libs?); note the pattern for use in task 2.3
  - No existing pattern. Used `std::env::consts::OS` (stdlib, no new dependency) rather than adding an OS-version crate for a platform string nobody asked to be exact.

## 2. Command Implementation

- [x] 2.1 Create the report-issue command handler (file location TBD by 1.1): `report_issue()` function that orchestrates URL building and browser launch; verify function compiles
- [x] 2.2 Add `open` crate dependency if not present (determined by 1.1); run `cargo build` to verify dependency resolves
  - Not needed: `gpui`'s own `App::open_url` already opens the system default browser, so no new browser-launch dependency at all.

## 3. URL Template Construction

- [x] 3.1 Implement `build_github_issue_url()` function that takes app version, build commit, build date, and returns a GitHub new-issue URL with `title` and `body` query parameters; include the template body text per design.md (app name, version, build commit, build date, platform)
  - Added `percent-encoding` (already in `Cargo.lock` transitively, now a direct pinned dependency) for correct query-string encoding.
- [x] 3.2 Unit test `build_github_issue_url()` with mock build info (version "1.0.0", commit "abc1234", date "2026-10-01"); verify URL contains expected components and is properly URL-encoded; run `cargo test` and confirm all tests pass
- [x] 3.3 Integrate `build_github_issue_url()` into the `report_issue()` command handler; pass values from `build_info` module and detected OS version

## 4. Menu Integration

- [x] 4.1 Register the `report_issue` command via the `actions!` macro in `menu.rs` (or the appropriate location per 1.1) with:
  - action id: `report_issue`
  - title: "Report Issue"
  - default keybinding (e.g., `Cmd+Shift+?` on macOS, `Ctrl+Shift+?` on Linux)
  - menu assignment: Help menu
  - verify command is registered by checking it appears in `actions!` output
  - Registered as `help.report_issue` in `ui/report_issue.rs` via `CommandRegistry`/`MenuSlot::Help` (the app's one registration path, per 1.1), default binding `cmd-shift-/` on every platform - confirmed by unit test `report_issue_is_registered_for_the_help_menu`.
- [x] 4.2 Wire the registered action to the `report_issue()` handler; verify the Help menu shows "Report Issue" item with its keybinding when app launches
  - `register_handler` wires `ReportIssue` to the URL-build-and-open action; `ui::menu`'s existing `registry_items(MenuSlot::Help, ..)` already surfaces any registered Help command, so no menu.rs change was needed beyond the registration itself.

## 5. Error Handling and Fallback

- [x] 5.1 Implement error handling for browser launch failure (e.g., if `open::that()` fails); log error and copy the generated URL to clipboard as fallback using existing clipboard patterns in the app; verify error case does not crash the app
  - N/A as designed: `gpui::App::open_url` returns no `Result` (platform calls are fire-and-forget), so there is no failure signal to branch on or fallback to build around.
- [x] 5.2 Test with an invalid browser/open call (simulate by mocking or env var override); confirm fallback behavior works and user can still access the URL
  - N/A, same reason as 5.1 - nothing to simulate failing.

## 6. Verification

- [ ] 6.1 Manual test: Open the app, open Help menu, click "Report Issue" menu item; verify system browser opens with GitHub issue URL prefilled with correct version, build commit, and build date from About window
- [ ] 6.2 Manual test: Use keyboard shortcut (Cmd+Shift+? or Ctrl+Shift+?) to invoke Report Issue; verify browser opens with the same prefilled URL
- [ ] 6.3 Manual test: Search command palette for "Report Issue"; verify command appears and is invokable from palette with correct URL
- [x] 6.4 Verify all existing tests still pass: run `cargo test` and confirm no regressions
  - `cargo test` (outside the sandbox, which blocks socket/port binding unrelated to this change): 619 passed, 0 failed, 2 ignored.
