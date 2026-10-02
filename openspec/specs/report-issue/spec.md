## Purpose

Provides users a quick, discoverable way to report bugs directly from the Help menu with prefilled app build information.

## Requirements

### Requirement: Report Issue menu item is discoverable and accessible

The application SHALL provide a "Report Issue" menu item in the Help menu that is keyboard-accessible as a registered command in the command palette.

#### Scenario: Report Issue appears in Help menu

- **WHEN** the application finishes starting up
- **THEN** the Help menu contains a "Report Issue" menu item

#### Scenario: Report Issue is in command palette

- **WHEN** user opens the command palette
- **THEN** the "Report Issue" command appears in the results

#### Scenario: Report Issue has keyboard shortcut

- **WHEN** user opens the keymap viewer or Help menu
- **THEN** the "Report Issue" menu item shows its assigned keyboard shortcut

### Requirement: Report Issue opens prefilled GitHub issue URL

When the user activates the Report Issue command (via menu, palette, or keyboard), the system SHALL open the default web browser with a GitHub new-issue URL that includes the app's build information.

#### Scenario: Report Issue opens browser with prefilled data

- **WHEN** user activates the Report Issue command
- **THEN** the system opens the default web browser with a GitHub new-issue URL at `https://github.com/pilgrimagesoftware/Fernrohr-App/issues/new?`

#### Scenario: URL includes app version and build identifier

- **WHEN** the GitHub issue URL is constructed
- **THEN** the URL query parameters include:
  - `title`: "Bug report: [description]" (placeholder text)
  - `body`: a template including app version, build commit, build date, and sections for "Steps to Reproduce" and "Expected vs. Actual" behavior

#### Scenario: Build info is accurate and consistent with About window

- **WHEN** the user reports an issue
- **THEN** the build commit and build date in the Report Issue URL match the values shown in the About window

### Requirement: Build information is sourced from the same system as About window

The report-issue command SHALL reuse Fernrohr's existing `build_info` module, which is already established by the about-window feature.

#### Scenario: Report Issue uses existing build_info module

- **WHEN** the app builds
- **THEN** the report-issue command has access to the same `FERNROHR_BUILD_COMMIT` and `FERNROHR_BUILD_DATE` environment variables used by the about-window

#### Scenario: Report Issue falls back gracefully when build info is unavailable

- **WHEN** build information is not available (source tarball, missing git)
- **THEN** the report-issue URL includes a fallback marker (e.g., "unknown") rather than an empty field
