# Design

## Context

See proposal.md - Why. The report-issue feature is a command handler that builds and opens a
GitHub issue URL, seeded with app build information. This reuses the `build_info` module
established by the about-window feature (same `FERNROHR_BUILD_COMMIT` and `FERNROHR_BUILD_DATE`
env vars stamped at compile time).

Fernrohr's command registry already supports menu item registration; see `command_system` spec
and the `menu.rs` pattern for how to wire a new command into the Help menu.

## Goals / Non-Goals

**Goals:**
- Add a "Report Issue" command that opens a browser with a GitHub issue URL prefilled with app
  version, build commit, and build date.
- Keyboard-accessible via command palette and menu, matching Fernrohr's keyboard-first
  conventions.
- Reuse the existing `build_info` module to avoid duplicating build-time logic.

**Non-Goals:**
- No custom dialog/window for composing the issue (browser URL template is sufficient).
- No programmatic GitHub API calls (browser URL is the simplest cross-platform approach).
- No i18n for button labels or error messages (existing app uses plain literals; deferred until
  app-wide i18n infrastructure is added).

## Decisions

- **Command location**: `app/src/commands/report_issue.rs` (new file under a `commands/` module),
  following the pattern of other command handlers if they exist, or directly in `menu.rs` if
  command handlers are currently inline (to be determined by reading existing code).
  - The command is registered via the `actions!` macro and wired to the Help menu in `menu.rs`.
- **URL construction**: Build the GitHub new-issue URL with query parameters:
  - `title`: "Bug report: [description]" (placeholder)
  - `body`: Multi-line template including app name, version, build commit, build date, platform
    (OS version via `sys_info` crate or std; defer exact method to task phase), and sections for
    "Steps to Reproduce" and "Expected vs. Actual Behavior".
- **Browser launch**: Use the `open` crate (lightweight, cross-platform) to open the URL with the
  system default browser. If `open` isn't already a dependency, add it. If a different mechanism
  is already used elsewhere in the app (e.g., for clicking external links), reuse that instead.
- **Error handling**: If the browser launch fails, log the error and optionally show a toast
  notification or copy the URL to clipboard as a fallback (exact UX to be determined during
  implementation based on existing error-handling patterns).
- **Keybinding**: Assign a sensible default, e.g., `Cmd+Shift+?` on macOS / `Ctrl+Shift+?` on
  Linux (following the Help menu convention). Defer to task phase if platform-specific bindings
  are needed.

## Risks / Trade-offs

- [Platform differences in `sys-info` crate or OS detection] → Fetch OS version using platform
  libs already in use (if any); fallback to simplified version if exact version is unavailable.
- [Browser launch failure] → URL is still available in clipboard or can be displayed as fallback
  text for copy-paste.
- [GitHub issue template evolving] → The template is a simple string built at runtime, so it's
  easy to update without recompilation if needed later.

## Open Questions

- Is there an existing `commands/` module or pattern in the app for command handlers, or are they
  currently inline in `menu.rs`?
- Does the app already depend on the `open` crate, or should we add it?
- Are there existing patterns for detecting the OS version and Kubernetes version (if a cluster
  is connected) to include in the bug report template?
