# Design

## Context

`typography` names Manrope for all UI text and Monaco for code; Manrope ships in
`app/assets/fonts/`. Spacing is set inline per panel with GPUI's `px`/`gap_*`/`p_*` helpers. Text
sizes come from gpui-component's theme and per-element `text_sm()`-style calls. The Settings window
and the command registry exist, and the preference file already holds the theme preference.

## Goals / Non-Goals

**Goals:**
- A frame/data/code type system that reads like a designed product.
- One spacing scale, roomier than today, and one text-size knob that scales both.

**Non-Goals:**
- No new colours or theme changes (`visual-language`'s colour rules stand).
- No per-panel or per-role size settings; one knob scales everything.
- No custom font picker in this change.

## Decisions

- **Adamina for the frame, Manrope for data.** Adamina is a calm serif designed for small sizes,
  and gives the frame a distinct voice; Manrope stays where legibility of dense values matters.
  Adamina ships one weight (Regular), so frame hierarchy comes from size and colour, not bold. Where
  emphasis needs weight (the selected tab, say), it uses colour from `visual-language`.
- **Text size is a scale factor**, not absolute sizes: the theme's base sizes multiply by a factor
  in fixed steps (e.g. 85%-150% in 10% steps, default 100%). Spacing tokens multiply by the same
  factor, so a larger size doesn't crowd.
- **Spacing tokens live in one module** (`ui::space`): `panel_inset`, `card_padding`, `row_height`,
  `section_gap`, `control_gap`. Rendering code calls these instead of literal pixels; a grep for
  literal `px(` in `ui/` that isn't a token is the review check.
- **Rendering quality is investigated before it's specified in detail.** Candidates: Manrope's
  variable-font weight axis rendering thinner than expected, tight line height, sizes landing on
  half pixels, and GPUI's glyph rasterization mode. The investigation renders the same strings in
  GPUI and in a web view, changes one variable at a time, and records which ones close the gap. The
  fix is whichever ones do; the requirement stays at the observable level.
- **Commands**: `view.increase_text_size` (⌘=), `view.decrease_text_size` (⌘-),
  `view.reset_text_size` (⌘0), global key context, in the View menu, and in the palette. Settings
  shows the same value with a stepper that's keyboard-operable.

## Risks / Trade-offs

- [Adamina has no bold or italic] → Hierarchy by size and colour; data stays in Manrope, which has
  the weights.
- [Touching spacing in every panel is a wide diff] → Land the token module and the font roles
  first, then convert panels in groups, each with its own render tests.
- [A serif in dense chrome might feel heavy at small sizes] → The manual check compares the
  context bar and tabs at 85% and 100% before sign-off.
