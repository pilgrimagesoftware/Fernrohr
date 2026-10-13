# Proposal

## Why

Report Issue is a modal dialog over the active window. It blocks the window a user is reporting
about, so they can't look up the detail they want to describe. It only collects a subject and a
description, with no way to say whether the report is a bug or a feature request, and no way to
include logs or screenshots. Knot's report form already offers those. It also opens with nothing
focused, so the user has to click into Subject before typing.

Fernrohr also has nowhere to read logs from. It logs through the `log` facade but never installs a
logger, so every warning it emits is discarded.

Tracks pilgrimagesoftware/Fernrohr-App#241 (a Knot-style report form in a separate non-modal window)
and pilgrimagesoftware/Fernrohr-App#249 (focus Subject when the form opens).

## What Changes

- Report Issue opens a separate, non-modal Report Issue window instead of a dialog. There is one at
  a time, and activating Report Issue again brings it forward. Subject has focus when it opens.
- A report is a **bug** or a **feature request**, labelled `bug` or `enhancement` on GitHub.
- A bug report can attach the end of the application log. This is opt-in and unchecked by default.
- A report can name up to five screenshots the user picks from disk. The issue body lists them, and
  Fernrohr shows the files so they can be dragged onto the issue, because `gh` can't upload them.
- The window is keyboard-operable through registered commands for Report and Close, and both
  buttons show their keys.
- Fernrohr writes an application log file, keeping the previous run's log beside it.

## Capabilities

### New Capabilities

- `app-logging`: Fernrohr writes its log records to a per-user log file, and keeps the previous
  run's file.

### Modified Capabilities

- `report-issue`: the dialog becomes a non-modal window, with a report type, an opt-in log
  attachment, screenshots, keyboard commands, and Subject focused on open.

## Impact

- App: `app/src/ui/report_issue*` is reworked from a dialog into a window. A new
  `app/src/util/app_log*` file logger is installed at startup, and `util::paths` gains the log
  directory. No new dependencies.
- Docs: the settings page's "Report an issue" section, and the keybindings reference for the new
  commands.
