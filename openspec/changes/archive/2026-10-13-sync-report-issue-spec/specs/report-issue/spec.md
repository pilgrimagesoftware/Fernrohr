# Spec Delta

## REMOVED Requirements

### Requirement: Report Issue opens prefilled GitHub issue URL

**Reason**: Report Issue no longer opens the browser directly. Since Fernrohr-App #152 it opens a
dialog, files through the GitHub CLI when it is ready, and uses the prefilled browser page only as a
fallback. The fixed "Bug report:" title and templated body are gone: the user writes the subject and
description.

**Migration**: The dialog is specified by "Report Issue opens a report dialog". The browser URL now
belongs to "Report falls back to a prefilled issue in the browser". The consistency with the About
window is covered by "Build information is sourced from the same system as About window".

## MODIFIED Requirements

### Requirement: Build information is sourced from the same system as About window

The report-issue command SHALL reuse Fernrohr's `build_info` module, which the About window uses, so
the dialog and the report show the same version and build identifier as the About window. The
dialog SHALL show the version, the build identifier and the platform, and the report body SHALL
append them below the user's description.

#### Scenario: Report Issue uses existing build_info module

- **WHEN** the app builds
- **THEN** the report-issue command has access to the same `FERNROHR_BUILD_COMMIT` and `FERNROHR_BUILD_DATE` environment variables used by the about-window

#### Scenario: Build info is consistent with the About window

- **WHEN** the user opens Report Issue
- **THEN** the version and the build identifier (build date and commit) match the values shown in
  the About window

#### Scenario: The report body carries the diagnostics

- **WHEN** a report is filed or opened in the browser
- **THEN** its body is the user's description, then a `---` rule, then `**Fernrohr version:**`,
  `**Build:**` and `**Platform:**` lines with this build's values, the platform being the operating
  system's name (for example `macos` or `linux`)

#### Scenario: Report Issue falls back gracefully when build info is unavailable

- **WHEN** the commit is not available (source tarball, missing git)
- **THEN** the build identifier reads `<build date>, commit unknown` rather than leaving the commit
  empty

## ADDED Requirements

### Requirement: Report Issue opens a report dialog

When the user activates the Report Issue command (via menu, palette, or keyboard), the system SHALL
open a Report Issue dialog over the active window with a Subject field, a Description field, this
build's diagnostics, a line saying how Report will file the issue, and Cancel and Report buttons.
Nothing SHALL be sent until the user chooses Report.

#### Scenario: Report Issue opens the dialog

- **WHEN** user activates the Report Issue command
- **THEN** the Report Issue dialog opens with empty Subject and Description fields and the build's
  version, build identifier and platform

#### Scenario: Report needs both fields

- **WHEN** the Subject or the Description is empty or only whitespace
- **THEN** Report is disabled

#### Scenario: Cancel sends nothing

- **WHEN** the user chooses Cancel
- **THEN** the dialog closes and no issue is filed or opened

#### Scenario: An error view prefills the report

- **WHEN** the user chooses Report… under a panel's error, or under a failed connection in the
  cluster picker
- **THEN** the Report Issue dialog opens with the Subject set to the error message's first line,
  cut to 80 characters with an ellipsis when longer, and the Description set to the message followed
  by the error detail in a fenced code block

#### Scenario: A connection failure report leaves the context out

- **WHEN** the dialog is prefilled from a failed connection in the cluster picker
- **THEN** the prefilled text names the failure and its reason but not the context's name

### Requirement: Report files the issue through the GitHub CLI when it is ready

As the dialog opens, the system SHALL check off the main thread whether `gh` is on the user's
login-shell `PATH` and signed in to github.com, and the dialog SHALL say which way Report will go.
When `gh` is ready, Report SHALL file the issue with `gh issue create` against
`pilgrimagesoftware/Fernrohr-App`, using the Subject as the title and the report body as the body.

#### Scenario: The dialog says how Report will file

- **WHEN** the dialog opens
- **THEN** it says it is checking for the GitHub CLI, then either that Report files the issue with
  the GitHub CLI or that Report opens a prefilled issue in the browser

#### Scenario: Report files through gh

- **WHEN** `gh` is ready and the user chooses Report
- **THEN** Report is disabled and reads "Reporting…" while `gh` runs, and when the issue is filed
  the dialog closes and a notification titled "Issue reported" shows the new issue's URL

### Requirement: Report falls back to a prefilled issue in the browser

When `gh` is not ready, or the check has not finished, Report SHALL open the default browser on
`https://github.com/pilgrimagesoftware/Fernrohr-App/issues/new` with the Subject as the `title`
parameter and the report body as the `body` parameter, both percent-encoded, and close the dialog.
When filing through `gh` fails, the system SHALL open the same page and say why `gh` failed.

#### Scenario: gh is not available

- **WHEN** `gh` is not installed or not signed in, and the user chooses Report
- **THEN** the browser opens on the prefilled new-issue page and the dialog closes

#### Scenario: gh fails to file the issue

- **WHEN** `gh issue create` fails
- **THEN** the dialog closes, the browser opens on the prefilled new-issue page, and a warning
  notification gives the reason `gh` reported
