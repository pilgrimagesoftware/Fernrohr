# Proposal

## Why

Panel controls (close, zoom, the ellipsis menu) appear as one shared row rather than on each
panel's own tab. This is not an app-level bug: `gpui-component` 0.6.6's `TabPanel` (the dock
library this app depends on) draws exactly one toolbar per tab group, reflecting only the
currently *active* tab's `toolbar_buttons`/`title_suffix`, and buries "Close" as a menu item
inside that shared toolbar's ellipsis dropdown rather than giving each tab strip entry (`Tab` in
`dock/tab_panel.rs`) its own close affordance. Confirmed by reading `render_toolbar`/`render_tabs`
in the vendored crate source directly - there is no builder flag or per-panel override that
changes this; it is how the component is built.

## What Changes

Nothing yet - this proposal documents the investigation and lays out the real options, since the
fix is not free the way the other feedback items were:

- **Option A - upgrade `gpui-kit` 0.6.6 -> 0.7.0.** Unknown whether the newer `gpui-component` it
  pulls in changes this behavior; needs checking the 0.7.0 changelog/source before committing to
  it. High blast radius: every panel, the dock skin, and the picker are built against 0.6.6's
  exact API surface, and this repo has already paid once for a dependency-version mismatch
  (`rust-i18n`/`rust-i18n-macro` drift during the `code-reorg` reconciliation).
- **Option B - patch the vendored crate.** Fork `gpui-component` (or vendor a local patch) to add
  a close control to `Tab` itself. Full control, but takes on maintaining a fork against upstream.
- **Option C - live with the shared toolbar.** Many multi-tab apps do exactly this (a toolbar for
  the active tab, tabs closed via the tab strip's own convention, here presently `ClosePanel`
  buried in the ellipsis menu, plus keyboard/command-palette close). Cheapest, but does not
  address the reported complaint.

## Capabilities

### Modified Capabilities
(none yet - no code changes in this proposal; a follow-up proposal implements whichever option is
chosen)

## Impact

None yet. Decision needed on Option A vs B vs C before any implementation proposal is written.
