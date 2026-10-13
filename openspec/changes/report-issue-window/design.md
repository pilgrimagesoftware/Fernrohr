# Design

## Context

Today Report Issue is a gpui-kit dialog (`app/src/ui/report_issue.rs`). It is filed through `gh`
when `gh` is ready, and through a prefilled browser page otherwise. Knot's bug report form
(`crates/knot/src/bug_report/`) is the model for the fields: a bug or feature type, opt-in log
tails, and picked screenshots. Knot hosts that form in a modal dialog. #241 asks for a separate
non-modal window instead.

## Goals / Non-Goals

**Goals:**

- A Report Issue window that stays open beside the main windows while the user looks things up.
- A report type, opt-in logs, and screenshots, with the same choices as Knot.
- Subject focused when the window opens (#249).
- An application log file worth attaching.

**Non-Goals:**

- Capturing screenshots. The user picks files, as in Knot. Capturing needs screen-recording
  permission on macOS and differs per Linux compositor.
- Uploading attachments. `gh issue create` can't, and the browser page can't be pre-filled with
  them.
- Redacting logs. The checkbox is off by default and its hint says what logs can contain.
- Log viewing in the app, log levels in settings, and size-based rotation.

## Decisions

### A separate window, one at a time

The window opens through `gpui_kit::open_window`, like About, and keeps its handle in the action
handler. Activating Report Issue again raises the open window rather than opening a second one. The
window is resizable, with a minimum size that fits the form. Closing it, by the window's close
control or the Close command, discards the report. Sending closes it.

The window raises because a second report window would split a report across two forms. That's the
same reasoning Knot uses for its single dialog.

### Error prefill never overwrites typed text

Report… under an error opens the window prefilled. If the window is already open and its Subject
and Description are both empty, the error fills them. Otherwise the window is only raised, so a
report in progress isn't replaced.

### Report type drives the label, the placeholders and the log offer

The type is a radio group with Bug (the default) and Feature request. A bug is labelled `bug`, and a
feature request `enhancement`. Switching the type keeps the typed text and changes only the
placeholders, and only a bug offers the log. If `gh` rejects the label, because most reporters can't
label issues in the repository, the issue is filed again without it. The browser fallback passes the
label as the `labels` parameter.

### Logs: the end of the current run's log, inside the body

The application log is attached as its last 200 lines, capped at 24 KiB, in a collapsed
`<details>` block with a four-backtick fence. That stays inside GitHub's body limit, and the fence
survives log lines that contain backticks. An unreadable log is named by its path instead of failing
the report. The browser fallback can't carry it within URL limits, so it asks the reporter to attach
the file by path.

### Screenshots: named, then shown

Add Screenshots… opens the system file picker for PNG, JPEG and GIF files, up to five. The issue
body lists their file names under a "Screenshots" heading. After the report is handed off, Fernrohr
shows the files so the user can drag them onto the issue. On macOS each file is revealed in Finder
(`open -R`). On Linux each distinct folder is opened (`xdg-open`). When the report was filed through
`gh` with screenshots, the new issue also opens in the browser, because that's where they're dropped.

### Notifications go to a main window

The report window closes when the report is sent, so the outcome ("Issue reported" or the `gh`
failure warning) is pushed to the active main window instead.

### Keyboard

Report Issue registers two commands in the window's `ReportIssueWindow` key context: Report
(`secondary-enter`), and Close (`escape`). Enter inside Description stays a newline. Both buttons
show their keys from the live keymap. Tab moves through type, Subject, Description, the log
checkbox, Add Screenshots…, each screenshot's remove button, Close and Report.

### Application log file

A small `log::Log` implementation writes records to `fernrohr.log` in the log directory:
`~/Library/Logs/com.pilgrimagesoftware.fernrohr/` on macOS, and `$XDG_STATE_HOME/fernrohr/` on Linux,
falling back to the cache directory. At startup the previous file becomes `fernrohr.log.1`. The level
is `info`, unless `RUST_LOG` names a level. Records also go to stderr when stderr is a terminal, so
`cargo run` still shows them. Tests don't install the logger.

## Risks / Trade-offs

- [Logs can contain context names, hosts or paths] -> The checkbox is off by default, and its hint
  says what logs can contain.
- [A non-modal window can outlive the window it reports about] -> The report doesn't refer to a
  window, and an error prefill copies its text when Report… is clicked.
- [`gh` rejects the label] -> File again without it.
