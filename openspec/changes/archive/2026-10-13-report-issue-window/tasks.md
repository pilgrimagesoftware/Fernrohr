# Tasks

## 1. Application log

- [x] 1.1 Add `util::paths::log_dir()`: `~/Library/Logs/<app>` on macOS, `dirs::state_dir()/<app>`
  on Linux, falling back to the cache directory. Verify: a unit test checks it ends with the app
  directory name.
- [x] 1.2 Add `util::app_log`, a `log::Log` writing timestamped records to `fernrohr.log`. Keep the
  previous file as `fernrohr.log.1`, honour `RUST_LOG` levels (default `info`), and echo to stderr
  when it is a terminal. Install it at startup outside tests. Verify: unit tests cover rotation,
  level parsing and the record format against a temp directory.
- [x] 1.3 Add a tail reader that returns the last 200 lines, at most 24 KiB, and drops a partial
  first line. Verify: unit tests cover short files, long files and the byte cap.

## 2. Report Issue window

- [x] 2.1 Replace the dialog with a single-instance `ReportIssueWindow` opened through
  `gpui_kit::open_window`. A second activation raises it. Subject is focused on open (#249). Verify:
  a test opens it twice and finds one window with Subject focused.
- [x] 2.2 Add the Bug and Feature request type radio group, with Bug as the default.
  Type-specific placeholders keep the text when the type switches. Verify: tests cover the default
  and that switching keeps the text.
- [x] 2.3 Add the "Attach the application log" checkbox, shown for bugs only and unchecked by
  default, with its hint. Verify: a test checks it is hidden for feature requests and that the body
  carries the log only when it is checked.
- [x] 2.4 Add Add Screenshots… through `prompt_for_paths`: PNG, JPEG and GIF, up to five, with
  duplicates ignored, the skipped files counted, and each file removable. Verify: unit tests cover
  the filter and the limit, and a test removes one.
- [x] 2.5 Build the body: description, Screenshots, Logs (with a four-backtick fence in
  `<details>`, or the path for the browser fallback or an unreadable log), then the diagnostics with
  the architecture. Verify: unit tests cover each combination.
- [x] 2.6 File through `gh` with `--label bug|enhancement`, retrying without the label on failure.
  The browser fallback adds the `labels` parameter. Notifications go to a main window. Verify: tests
  with the fake `gh` cover the labelled filing, the unlabelled retry and the fallback.
- [x] 2.7 After hand-off, show the screenshots (`open -R` on macOS, `xdg-open` per folder on Linux),
  and open the new issue when it was filed through `gh` with screenshots. Verify: a unit test checks
  the commands built for each platform.
- [x] 2.8 Keep error prefill: `ReportError` opens the window prefilled, or raises an open window
  without overwriting typed text. Verify: tests cover both cases.
- [x] 2.9 Register Report (`secondary-enter`), Close (`escape`) and Add Screenshots…
  (`secondary-shift-a`) in the `ReportIssueWindow` key context, and show Close's and Report's keys on
  the buttons. Verify: tests send the keystrokes and check that the
  report is sent and the window closed.

## 3. Docs

- [x] 3.1 Rewrite the settings page's "Report an issue" section for the window, the report type,
  logs and screenshots, and say where the log file lives. Update the error Report… subsection.
  Verify: `bundle exec ruby _scripts/check.rb` passes.
- [x] 3.2 Add the new commands to `reference/keybindings.md`. Verify: they match the registry.

## 4. Validation

- [x] 4.1 Run `cargo fmt -- --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test`.
  Verify: all pass.
