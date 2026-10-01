# Proposal

## Why

Users discovering bugs in Fernrohr need a quick path to report them. Currently, there's no
discoverable link or prefilled template in the UI. The about-window shows the build identifier
(commit + date), but filing a bug requires switching to a browser, navigating to GitHub, and
manually typing the version and build info. A dedicated "Report Issue" menu item that opens a
prefilled GitHub issue template in the browser, seeded with the app's build info, closes that gap
and makes bug reporting frictionless.

## What Changes

- Add a "Report Issue" menu item to the Help menu, wired to a registered command.
- The command opens the system default browser with a prefilled GitHub new-issue URL
  (`https://github.com/pilgrimagesoftware/Fernrohr-App/issues/new?`) including:
  - Title placeholder: "Bug report: [description]"
  - Body template: app version, build commit, build date, platform/OS version, Kubernetes version
    (if a cluster is connected), and a section for "Steps to Reproduce" and "Expected vs. Actual".
- The implementation reuses Fernrohr's `build_info` module (already established in `about-window`)
  to stamp build data; no new build-time infrastructure needed.
- The menu item is keyboard-accessible as a command-palette entry and has a default keybinding
  (e.g., `Cmd+Shift+?` or a Help-menu-scoped binding).

## Capabilities

### New Capabilities

- `report-issue`: the "Report Issue" menu item, command registration, browser launch, and GitHub
  issue URL template construction.

### Modified Capabilities

- `application-menu`: adds a new menu item to the Help menu; the menu structure and sourcing
  behavior unchanged, only the Help menu's item list grows.

## Impact

- `app/src/ui/menu.rs`: register a `report_issue` command and wire it to the Help menu.
- `app/src/ui/report_issue.rs` or `app/src/commands/report_issue.rs` (new): command handler,
  URL template builder using `build_info`, browser launch (via `open::that()` or similar).
- No new dependencies if `open` crate isn't already present; if not, add it (lightweight,
  cross-platform browser/URL launcher).
- No changes to persistence, watcher, or cluster logic.

