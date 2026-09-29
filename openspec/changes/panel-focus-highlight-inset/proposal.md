# Proposal

## Why

`panel_title::focus_frame` draws the focused-panel border as a plain `.size_full().border_1()`
rectangle flush with the panel's own bounds. For a panel occupying the bottom of the window, that
border is drawn right up to where macOS clips the window content to its own rounded corner, so
the border's bottom corners get visibly cut off - a sharp rectangle colliding with a rounded mask.
Every panel using `focus_frame` (all of them - it's the app-wide, section-12 focus treatment)
shows this at the window's bottom edge.

## What Changes

- The focused-panel border no longer sits flush with the panel's outer edge; it's inset slightly
  so it clears the OS's window-corner rounding rather than being clipped by it.

## Capabilities

### Modified Capabilities
(none - this is a rendering correction to the existing section-12 focus treatment, not a new
behavior or a change to what is specified. `skip_specs: true` in `.openspec.yaml`.)

## Impact

- `app/src/ui/panel/title.rs`: `focus_frame`'s border/sizing.
