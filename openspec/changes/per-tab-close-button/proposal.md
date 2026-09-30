# Proposal

## Why

Two related reports trace to the same root cause: panel controls (close, zoom, the ellipsis menu)
appear as one shared row rather than on each panel's own tab, and there is no way to color an
individual tab to show which panel has focus. `gpui-component` 0.6.6's `TabPanel` (the dock
library this app depends on) draws exactly one toolbar per tab group, reflecting only the
currently *active* tab's `toolbar_buttons`/`title_suffix`/`title_style`, and its tab strip
(`render_tabs`) does not read `Panel::title_style` at all - only the single-panel, no-tab-strip
case (`render_title`) does. Confirmed by reading `render_toolbar`/`render_tabs`/`render_title` in
the vendored crate source directly, twice now (once for the close button, again while removing
the borrowed-content border in favor of "color the tab instead") - there is no builder flag or
per-panel override that reaches the tab strip; it is how the component is built.

> **Update (2026-09-30):** the focus-colour half of this is resolved by
> `panel-focus-highlight-inset`. `render_tabs` ignores `title_style`, but it draws the panel's
> own `title()` element as the label whenever `tab_name()` is `None` (every Fernrohr panel's is),
> so a panel can mark its own tab on 0.6.6. What stays out of reach is styling the `Tab` itself
> (background, a close control), so the close-button half below is unchanged.

## What Changes

Nothing yet - this proposal documents the investigation and lays out the real options, since the
fix is not free the way the other feedback items were:

- **Option A - upgrade `gpui-kit` 0.6.6 -> 0.7.0.** Unknown whether the newer `gpui-component` it
  pulls in changes this behavior; needs checking the 0.7.0 changelog/source before committing to
  it. High blast radius: every panel, the dock skin, and the picker are built against 0.6.6's
  exact API surface, and this repo has already paid once for a dependency-version mismatch
  (`rust-i18n`/`rust-i18n-macro` drift during the `code-reorg` reconciliation).
- **Option B - patch the vendored crate.** Fork `gpui-component` (or vendor a local patch) to add
  a close control and a focus-aware color hook to `Tab` itself. Full control (would fix both
  reports in one patch), but takes on maintaining a fork against upstream.
- **Option C - live with the shared toolbar, no per-tab focus color.** Many multi-tab apps do
  exactly this (a toolbar for the active tab, tabs closed via the tab strip's own convention, here
  presently `ClosePanel` buried in the ellipsis menu, plus keyboard/command-palette close; no
  visual distinction for "which panel has focus" beyond which tab is selected). Cheapest, but
  doesn't address either report.

The border removed from `focus_frame` (see the commit removing it) is not restored by any of
these options on its own - it was the wrong mechanism regardless of which option lands here, since
it lived in panel content, not the tab strip.

## Capabilities

### Modified Capabilities
(none yet - no code changes in this proposal; a follow-up proposal implements whichever option is
chosen)

## Impact

None yet. Decision needed on Option A vs B vs C before any implementation proposal is written.
