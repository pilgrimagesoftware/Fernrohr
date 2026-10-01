# Proposal

## Why

Next to a tool like Knot, Fernrohr looks cramped and its text rough. Every surface uses one sans
(Manrope), so panel chrome and the data inside it read as the same thing. Padding is set ad hoc per
panel, and mostly tight. Text size is fixed, so users who want denser or larger text can't adjust
it. The result works, but it isn't pleasant to read for long.

## What Changes

- **Three type roles instead of two.** A new *UI* font, Adamina (bundled, SIL OFL), for the
  application's frame: panel titles, tabs, section headers, buttons, menus, the context bar, dialogs
  and the status bar. Manrope stays, as the *data* font, for tables, field values, chips and cards.
  Monaco stays the *code* font for YAML and logs.
- **Spacing tokens.** One shared scale for panel insets, card padding, row height and section gaps,
  a step roomier than today, that every panel draws from instead of hard-coded pixels.
- **Configurable text size.** A text-size preference, defaulting to today's sizes, that scales all
  three roles together. It's adjustable in Settings and through Increase / Decrease / Reset Text
  Size commands (⌘= / ⌘- / ⌘0), and persisted in the preference file.
- **Smoother text rendering.** A short investigation finds out why GPUI-rendered text looks rougher
  than Knot's (font, size, line height, weight, or glyph rasterization), and the change applies
  the fix that makes the difference.

## Capabilities

### New Capabilities

(none)

### Modified Capabilities

- `typography`: the UI/data split (Adamina + Manrope), configurable text size, and the
  rendering-quality requirement.
- `visual-language`: spacing tokens with a minimum inset for panels and cards.

## Impact

- `App/app/assets/fonts/` (adds Adamina and its OFL), font registration and the theme's font
  settings.
- A spacing-token module used by panel, card, table and dialog rendering; touches most `ui/`
  files mechanically.
- Settings window, command registry and preference file (text size).
