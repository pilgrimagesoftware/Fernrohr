# Spec Delta

## REMOVED Requirements

### Requirement: Report Issue opens a report dialog

**Reason**: Report Issue opens a separate, non-modal window instead of a dialog, so the user can keep
working in the window they are reporting about (Fernrohr-App#241).

**Migration**: The window, its fields, its prefill from error views and its close behavior are
specified by "Report Issue opens a report window" and the requirements added beside it.

## MODIFIED Requirements

### Requirement: Build information is sourced from the same system as About window

The report-issue command SHALL reuse Fernrohr's `build_info` module, which the About window uses, so
the window and the report show the same version and build identifier as the About window. The
window SHALL show the version, the build identifier and the platform, and the report body SHALL
end with them, below the description and any screenshot and log sections.

#### Scenario: Report Issue uses existing build_info module

- **WHEN** the app builds
- **THEN** the report-issue command has access to the same `FERNROHR_BUILD_COMMIT` and `FERNROHR_BUILD_DATE` environment variables used by the about-window

#### Scenario: Build info is consistent with the About window

- **WHEN** the user opens Report Issue
- **THEN** the version and the build identifier (build date and commit) match the values shown in
  the About window

#### Scenario: The report body carries the diagnostics

- **WHEN** a report is filed or opened in the browser
- **THEN** its body is the user's description, then a "Screenshots" section when screenshots were
  chosen, then a "Logs" section when the log is attached, then a `---` rule and `**Fernrohr
  version:**`, `**Build:**` and `**Platform:**` lines, the platform being the operating system and
  architecture (for example `macos aarch64`)

#### Scenario: Report Issue falls back gracefully when build info is unavailable

- **WHEN** the commit is not available (source tarball, missing git)
- **THEN** the build identifier reads `<build date>, commit unknown` rather than leaving the commit
  empty

### Requirement: Report files the issue through the GitHub CLI when it is ready

As the window opens, the system SHALL check off the main thread whether `gh` is on the user's
login-shell `PATH` and signed in to github.com, and the window SHALL say which way Report will go.
When `gh` is ready, Report SHALL file the issue with `gh issue create` against
`pilgrimagesoftware/Fernrohr-App`, using the Subject as the title, the report body as the body, and
the report type's label. If `gh` fails with the label, it SHALL file again without it.

#### Scenario: The dialog says how Report will file

- **WHEN** the Report Issue window opens
- **THEN** it says it is checking for the GitHub CLI, then either that Report files the issue with
  the GitHub CLI or that Report opens a prefilled issue in the browser

#### Scenario: Report files through gh

- **WHEN** `gh` is ready and the user chooses Report
- **THEN** Report is disabled and reads "Reporting…" while `gh` runs, and when the issue is filed
  the window closes and a notification titled "Issue reported" shows the new issue's URL in a main
  window

#### Scenario: The label is refused

- **WHEN** `gh issue create` fails with the type's label
- **THEN** the issue is filed again without a label

### Requirement: Report falls back to a prefilled issue in the browser

When `gh` is not ready, or the check has not finished, Report SHALL open the default browser on
`https://github.com/pilgrimagesoftware/Fernrohr-App/issues/new` with the Subject as the `title`
parameter, the report body as the `body` parameter and the type's label as the `labels` parameter,
all percent-encoded, and close the window. When filing through `gh` fails, the system SHALL open the
same page and say why `gh` failed.

#### Scenario: gh is not available

- **WHEN** `gh` is not installed or not signed in, and the user chooses Report
- **THEN** the browser opens on the prefilled new-issue page and the window closes

#### Scenario: gh fails to file the issue

- **WHEN** `gh issue create` fails, with and without the label
- **THEN** the window closes, the browser opens on the prefilled new-issue page, and a warning
  notification in a main window gives the reason `gh` reported

#### Scenario: The browser page can't carry the log

- **WHEN** a bug report with the log attached falls back to the browser
- **THEN** its Logs section names the log file's path and asks the reporter to attach it, instead of
  carrying the log's text

## ADDED Requirements

### Requirement: Report Issue opens a report window

When the user activates Report Issue, the system SHALL open a separate, non-modal Report Issue
window, with the report type, Subject, Description, the log option, screenshots, the build's
diagnostics, a line saying how Report will file, and Close and Report buttons. Only one SHALL be
open: activating Report Issue again SHALL bring the open window forward.

#### Scenario: Report Issue opens the window

- **WHEN** user activates Report Issue and no Report Issue window is open
- **THEN** a Report Issue window opens with Bug selected, empty Subject and Description, and Subject
  focused, and the other windows stay usable

#### Scenario: Report Issue is already open

- **WHEN** user activates Report Issue while the Report Issue window is open
- **THEN** that window comes to the front with its contents unchanged and no second window opens

#### Scenario: Report needs both fields

- **WHEN** the Subject or the Description is empty or only whitespace
- **THEN** Report is disabled

#### Scenario: Closing sends nothing

- **WHEN** the user chooses Close or closes the window
- **THEN** the window closes and no issue is filed or opened

### Requirement: Error views prefill the report window

Report… under a panel's error, or under a failed connection in the cluster picker, SHALL open the
Report Issue window with Bug selected, the Subject set to the error message's first line cut to 80
characters with an ellipsis when longer, and the Description set to the message followed by the
error detail in a fenced code block. It SHALL NOT replace a report the user has started.

#### Scenario: An error view prefills the report

- **WHEN** the user chooses Report… under an error and no Report Issue window is open
- **THEN** the window opens prefilled from the error

#### Scenario: A report is already in progress

- **WHEN** the user chooses Report… while the Report Issue window is open with text in Subject or
  Description
- **THEN** the window comes to the front with its text unchanged

#### Scenario: A connection failure report leaves the context out

- **WHEN** the window is prefilled from a failed connection in the cluster picker
- **THEN** the prefilled text names the failure and its reason but not the context's name

### Requirement: A report is a bug or a feature request

The window SHALL offer the report types Bug and Feature request, with Bug selected when it opens. A
bug SHALL be labelled `bug`, and a feature request `enhancement`. Switching the type SHALL keep the
typed Subject and Description.

#### Scenario: A feature request

- **WHEN** the user selects Feature request and reports
- **THEN** the issue is labelled `enhancement` and offers no log

#### Scenario: Switching type keeps the text

- **WHEN** the user has typed a Subject and Description and switches the type
- **THEN** both fields keep their text

### Requirement: A bug report can attach the application log

A bug report SHALL offer an "Attach the application log" option, unchecked when the window opens,
with a hint that logs can include context names, hosts and paths. When it is checked, the report
body SHALL carry the last 200 lines of the current log, at most 24 KiB, in a collapsed section.

#### Scenario: The log is attached

- **WHEN** the user checks the option on a bug report and reports through `gh`
- **THEN** the body's Logs section holds the end of `fernrohr.log` in a collapsed block

#### Scenario: The log can't be read

- **WHEN** the option is checked and the log file can't be read
- **THEN** the report is still sent, and its Logs section names the log's path instead

### Requirement: A report can name screenshots

The window SHALL let the user add up to five PNG, JPEG or GIF files through the system file picker,
list them by file name, and remove each one. The report body SHALL list the chosen file names under
a "Screenshots" heading. Once the report is handed off, the system SHALL show the files so they can
be dragged onto the issue.

#### Scenario: Screenshots are added

- **WHEN** the user chooses Add Screenshots… and picks image files
- **THEN** the window lists them by name, ignoring duplicates, and says how many it skipped over the
  limit of five or for not being an image

#### Scenario: A report with screenshots is filed

- **WHEN** a report with screenshots is filed through `gh`
- **THEN** the new issue opens in the browser and the screenshot files are shown, revealed in Finder
  on macOS and their folders opened on Linux

### Requirement: The report window is keyboard-operable

The window SHALL register Report (`secondary-enter`), Close (`escape`) and Add Screenshots…
(`secondary-shift-a`) as commands in its key context, and the Close and Report buttons SHALL show
their keys. Report's key SHALL work from Description too. Tab SHALL reach every control in reading
order, and Enter in Description SHALL insert a newline.

#### Scenario: Reporting from the keyboard

- **WHEN** both fields are filled and the user presses ⌘↵ (Ctrl+Enter off macOS)
- **THEN** the report is sent as if Report were clicked

#### Scenario: Reporting from Description

- **WHEN** focus is in Description and the user presses ⌘↵
- **THEN** the report is sent rather than a newline inserted

#### Scenario: Closing from the keyboard

- **WHEN** the user presses Escape in the window
- **THEN** the window closes and nothing is sent
