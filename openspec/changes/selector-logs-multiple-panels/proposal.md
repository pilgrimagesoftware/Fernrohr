# Proposal

## Why

The typed label Logs panel (Open Logs by Label…) allows one panel per cluster context. Opening it
again focuses the existing panel, and switching a panel to a context that already has one focuses
that panel instead. So a user can't keep `app=web` and `app=worker` streaming side by side in the
same cluster, or compare one selector across two contexts in separate panels. Each typed panel
should be its own instance.

## What Changes

- **BREAKING (behavior):** Open Logs by Label… always opens a new, empty typed panel instead of
  focusing an open one. It still opens in the focused list's context and namespaces, else the
  window's active context.
- Any number of typed panels can be open at once, in the same context or different ones. Each keeps
  its own selector, container filter, namespace scope and context.
- **BREAKING (behavior):** switching a typed panel's context always moves that panel to the chosen
  context, in its own tab, even when another typed panel already searches that context. It never
  focuses a different panel.
- Each typed panel is saved and restored separately: a saved layout holding several of them brings
  back each one with its own selector, filter, namespaces and context. Layouts saved before this
  change, with at most one typed panel per context, still restore.
- Titles and tabs stay distinguishable. A panel with a selector is named by it (`Logs: app=web`),
  as today. In a window holding several contexts the title also names the context, as other panels'
  titles do.
- Workload Logs panels (a Deployment's pods, for example) are unchanged: opening one again for the
  same workload still focuses it.

## Capabilities

### New Capabilities

(none)

### Modified Capabilities

- `selector-logs`:
  - opening the typed panel always opens a new one, and several can be open at once (replaces the
    one-per-context rule and its "Reopening focuses the panel" scenario);
  - switching context always moves the panel itself (drops the focus-the-other-panel rule);
  - saved layouts keep several typed panels apart, and older layouts still load.

## Impact

App-only (`App` submodule), no new dependencies, no backend or API changes:

- `app/src/util/logs/labels.rs` - each typed panel's identity, so the window keys every instance
  apart, and reading older saved state that has none.
- `app/src/util/shell/logs_open.rs` - Open Logs by Label… opens a new panel every time.
- `app/src/util/shell/logs_context.rs` - switching context no longer focuses another panel.
- `app/src/util/shell/panels.rs`, saved-layout restore - distinct keys for restored typed panels.
- Docs: `docs/site/latest/logs-shells.md`, "Follow pods by label": several panels, the reopen
  behavior and context switching.
