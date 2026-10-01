# Proposal

## Why

From the user's `resource-links` smoke test: "the look of the app is very two-tone, i.e.,
background color and text color (except for the 'chips') - it could use some more color and/or
definition, because it seems to be easy to get 'lost' in the presentation right now."

It is two-tone, and not by choice. Fernrohr applies no theme of its own. It runs on gpui-component's
default, which is deliberately monochrome: `primary` is neutral-900 on light and neutral-50 on dark,
the body text colour, and `link` is plain black or white. Across the app's views, colour comes from
`muted_foreground` (31 uses), `border` (9), and a handful of status colours. `accent` is used once.
The result: nothing tells a panel's header from its body, a selected row from its neighbours, or a
healthy pod from a failing one, except the text itself.

## What Changes

- **One accent, used for everything interactive and current.** The accent the focused tab already
  uses (the macOS accent colour, else the theme's blue) becomes the theme's `primary`, `ring`,
  `link` and selection colour. Selected rows, primary buttons, focus rings, links and the active tab
  then share one hue, distinct from text, in light and dark mode.
- **Surfaces in layers, by fill rather than frame.** Three levels: the window background; panel
  chrome (each panel's heading and hint bar, table headers, the Resource panel's section headers)
  on a slightly raised fill; and cards inside detail views (containers, events, configuration) on a
  fill of their own. Paul rejected borders framing panel content (`panel-focus-highlight-inset`), so
  this adds no new borders around panel bodies. Definition comes from fills.
- **Status reads as colour at a glance.** The Pods table's Status cell is colour-coded (running
  green, pending amber, failed or crash-looping red, completed muted), Ready shows a coloured dot,
  and a non-zero restart count is amber. Condition badges and event reasons keep today's tones,
  from the same palette.
- **Detail views get structure.** Section headings get an accent bar and heavier weight, structured
  rows alternate a faint stripe, and the label column is quieter than its values.
- **Named semantic tokens, in one module.** `ui::style` exposes `accent`, `accent_subtle`,
  `surface_raised`, `surface_card`, `stripe` and `status(Tone)`. Views ask for those rather than raw
  theme fields, so the next view is consistent by default, and a later theme change lands in one
  place.
- **Contrast is a requirement, not a hope.** Text on every surface meets WCAG AA (4.5:1), and the
  accent and status colours meet 3:1 against their backgrounds, in both modes, checked by tests.

## Capabilities

### New Capabilities
- `visual-language`: the app's accent, surface layers, status colours and contrast floor.

## Impact

- `app/src/ui/theme.rs`: overrides a handful of theme tokens after each theme change (next to where
  fonts and the accent are already reasserted).
- `app/src/ui/style.rs` (new): the semantic tokens and the contrast helper.
- Panels' render code (pods table, pod and object detail, Resource panel, hint bars, `ui::detail`):
  swaps raw theme fields for tokens. No layout change.
- No new dependencies.
