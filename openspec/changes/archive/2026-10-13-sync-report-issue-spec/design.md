# Design

## Context

`report-issue` was written for the first, browser-only Report Issue. The app has since gained a
dialog and `gh` filing (Fernrohr-App #152) and error-prefilled reports (#177), and the spec wasn't
updated. This change brings the spec in line with the code on `develop`
(`app/src/ui/report_issue.rs`, `app/src/ui/report_issue/gh.rs`,
`app/src/ui/about_window/build_info.rs`, `app/src/ui/panel/title.rs`, `app/src/ui/picker/render.rs`).

## Goals / Non-Goals

**Goals:**

- The spec describes what a user sees and gets: the dialog, its diagnostics, how Report chooses
  between `gh` and the browser, and what each route reports back.
- Every requirement can be checked against the code.

**Non-Goals:**

- No behavior change. Where the spec and the code disagree, this change takes the code's side.
- No App changes, including docs. A docs mismatch is flagged as a separate App fix.

## Decisions

### Replace the "opens prefilled GitHub issue URL" requirement

That requirement's name and scenarios describe the old flow: a fixed "Bug report:" title and a
templated body opened straight in the browser. A MODIFIED block can't drop scenarios, so the
requirement is REMOVED, with a reason and migration, and "Report Issue opens a report dialog" is
ADDED. The URL details move to the new browser-fallback requirement, where they still apply.

### Split filing and fallback into their own requirements

Filing through `gh` and falling back to the browser are separate behaviors with separate triggers.
Each gets its own requirement and scenarios, which keeps every description under the strict length
limit.

### Keep the build-information requirement's name

The requirement still holds: the dialog and the report reuse the About window's build information.
It is modified to name the fields (version, build identifier, platform), the body layout, and the
`commit unknown` fallback. The build date is always stamped, so only the commit can be unknown.

## Risks / Trade-offs

- [The spec drifts again as Report Issue changes] -> The settings page names `report-issue`, and
  the project rules require a spec and docs update with each user-facing change.
