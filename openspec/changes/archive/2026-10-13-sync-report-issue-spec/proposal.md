# Proposal

## Why

The `report-issue` spec says Report Issue opens the browser on a prefilled GitHub new-issue URL,
with a "Bug report:" title and a templated body. The app no longer works that way. Since Fernrohr-App
#152 and #177, Report Issue opens a dialog with Subject and Description fields and the build's
diagnostics. Report files the issue through the GitHub CLI (`gh`) when it is installed and signed
in, and opens the prefilled browser page otherwise or when `gh` fails. Error views open the same
dialog with the error filled in. The spec is the contract the docs site restates, so it has to
describe what ships.

## What Changes

- Replace the requirement that Report Issue opens a prefilled URL with one that it opens a dialog.
- Rewrite the build-information requirement to describe the diagnostics the dialog shows and the
  report body carries.
- Add requirements for filing through `gh` and for the browser fallback.
- No behavior change. Every requirement restates the code on `develop`.

## Capabilities

### New Capabilities

None.

### Modified Capabilities

- `report-issue`: describes the shipped dialog, `gh` filing and browser fallback instead of the
  original browser-only flow.

## Impact

- Meta repo only: `openspec/specs/report-issue/spec.md`.
- No App code changes, so no App bump. The docs site's settings page already describes the dialog;
  any mismatch found there is an App docs fix, tracked separately.
