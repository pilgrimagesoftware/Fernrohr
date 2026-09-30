# Design

## Context

- `ui::theme` applies the light/dark preference and then reasserts fonts and the accent after
  every appearance change. `ui::accent` resolves the accent (macOS `controlAccentColor`, else the
  theme's `blue`) into a global. Both already sit on the path every theme change takes.
- The dock's tab strip and title bar are drawn by gpui-component from theme tokens, so tokens set
  on `Theme` reach them without forking the library. Panel bodies are the app's own code.
- A border framing panel content was tried and rejected (`panel-focus-highlight-inset`, Paul's
  review 2026-09-29).

## Goals / Non-Goals

**Goals:** one accent; three surface levels; status colour; structure in detail views; a contrast
floor that tests enforce; tokens that make the next view consistent.

**Non-Goals:**
- User-selectable themes or accent pickers. The macOS accent is already the user's choice; a theme
  picker is its own change.
- New icons, illustrations or motion.
- Changing layout, spacing or typography families (`typography` stays as specified).

## Decisions

### Override tokens rather than ship a theme file

Six theme fields are set after each theme change: `primary`/`primary_foreground`, `ring`, `link`,
`list_active`/`table_active`, and `accent`/`accent_foreground` (the row-hover fill). Everything
else stays gpui-component's. Why not a full JSON theme: a theme file pins every colour, including
the ones the library tunes between versions, for the sake of the handful that matter here. If the
overrides grow past a dozen, revisit.

### Surfaces are derived from the background, not picked

`surface_raised` and `surface_card` are the background shifted by a small lightness step (lighter
in dark mode, darker in light mode), and `stripe` is half of that step. Deriving them keeps them in
step with whatever background the mode has, and keeps the contrast test simple.

### Status colours come from one table

`style::status(Tone)` maps `Good`, `Warning`, `Bad` and `Neutral` to the theme's `success`,
`warning`, `danger` and `muted_foreground`. `ui::detail::BadgeTone` gains `Bad`. Pod phase to tone:
Running and Succeeded are Good and Neutral respectively; Pending is Warning; Failed and any
`CrashLoopBackOff`/`Error`/`ImagePullBackOff` waiting reason are Bad. Deciding tone is the
projection's job and is tested there; the renderer only maps tone to colour.

### Contrast, checked

`style::contrast(a, b)` computes the WCAG ratio. A test renders nothing: it builds each mode's
theme with the overrides applied and asserts text at 4.5:1 on every surface, and accent and status
colours at 3:1, on their backgrounds. A user accent that fails (a very light macOS accent in light
mode) is darkened in steps until it passes, so the system colour is respected as far as contrast
allows.

## Risks / Trade-offs

- [Subjective: "more colour" can overshoot] → The accent is one hue, status colours appear only
  where there is a status, and surfaces are small lightness steps. Section 5 of the tasks is a
  before/after review with the user before the change is done.
- [gpui-component retunes its defaults] → Only six tokens are pinned; the rest follows the library.
- [Striping reads as noise in short lists] → Only structured detail rows stripe; tables use the
  selection tint and row hover only.
