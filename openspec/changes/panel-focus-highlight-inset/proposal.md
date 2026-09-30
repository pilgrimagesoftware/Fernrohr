# Proposal

## Why

The app's focused-panel indicator started life as a border that `panel_title::focus_frame` drew
around each panel's content. It had two problems in turn:

1. Drawn flush with the panel's edge, its square corners got clipped by macOS's rounded window
   mask at the bottom of the window. This change first fixed that by insetting the border.
2. Paul's review (2026-09-29) then rejected the border itself as unwanted "inner framing" around
   panel content, and said the indicator belongs on the panel's tab instead. The border was
   removed (App `198dcaa`), leaving no focus indicator at all. The tab version was filed under
   `per-tab-close-button` as not reachable on gpui-component 0.6.6.

That last claim was only half right. The tab strip (`TabPanel::render_tabs`) does not read
`Panel::title_style`. But when a panel's `tab_name()` is `None`, which every Fernrohr panel's is,
the strip draws the panel's own `title()` element as the tab label. `title()` runs with the
window, so it can ask whether its panel has focus. A panel can therefore mark its own tab on
0.6.6, without upgrading or forking the library.

The change keeps its original directory name so its history stays traceable; its scope is now the
focus indicator as a whole, not only the inset.

## What Changes

- The focused panel's tab label (and the single-panel title bar, which draws the same element)
  carries a 2px underline in the user's accent colour (macOS; the theme's blue elsewhere) while
  keyboard focus is anywhere inside the panel. Unfocused tabs reserve the same 2px, drawn transparent, so labels never shift as focus
  moves.
- Placeholder and Logs panels now track their focus handle on their content. Before this, a click
  could not focus them, so their tab could never be marked.
- The no-op `focus_frame` wrapper left behind by the border's removal is deleted.

## Input routes

- **Mouse:** clicking inside a panel, or on its tab, focuses it and moves the underline there.
- **Keyboard:** the underline follows focus wherever a key moves it (for example into a panel's
  own controls). There is not yet a keyboard route that moves focus *between* panels: the dock
  offers only `ToggleZoom` and `ClosePanel`, and opening a panel does not focus it. That is a
  missing command, not part of this indicator, and is recorded as a follow-up in `tasks.md`.

## Capabilities

### Modified Capabilities
(none - a rendering change to the existing section-12 focus treatment. `skip_specs: true` in
`.openspec.yaml`.)

## Impact

- `app/src/ui/panel/title.rs`: `title_element` takes the focus state and draws the underline;
  `focus_frame` removed.
- `app/src/ui/accent.rs` (new): the system accent colour, with its fallback and refresh points.
  `app/Cargo.toml`: `objc2-app-kit =0.3.2` as a direct macOS-only dependency.
- `app/src/ui/placeholder.rs`, `app/src/util/logs.rs`: track their focus handle.
- `app/src/k8s/resource/pods.rs`, `app/src/k8s/resource/pod_detail.rs`: pass focus to
  `title_element`, drop `focus_frame`.
- `openspec/changes/per-tab-close-button`: its focus-colour half is resolved here. Its close-button
  half is unchanged and still blocked on the A/B/C decision.
