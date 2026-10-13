# Tasks

## 1. Spec

- [x] 1.1 Write the `report-issue` delta from the code on `develop`: the dialog and its prefill from
  error views, the diagnostics and body layout, `gh` readiness and filing, and the browser fallback.
  Verify: each requirement and scenario matches `app/src/ui/report_issue.rs`,
  `app/src/ui/report_issue/gh.rs`, `app/src/ui/about_window/build_info.rs`,
  `app/src/ui/panel/title.rs` and `app/src/ui/picker/render.rs`.
- [x] 1.2 Run `openspec validate sync-report-issue-spec --strict`. Verify: it passes.

## 2. Docs check

- [x] 2.1 Compare the "Report an issue" section of `docs/site/latest/settings.md` on App `develop`
  with the delta. Verify: the dialog, fields, route line, `gh` filing and browser fallback match.
  The two gaps found are flagged as an App docs fix, not changed here: the page says the build
  commit and date show `unknown`, though only the commit can be unknown (`<date>, commit unknown`),
  and no page documents the Report… button on error views.
