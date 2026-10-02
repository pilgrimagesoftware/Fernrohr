# Tasks

## 1. Rendering investigation

- [x] 1.1 Render the same strings (a table row, a panel title, a chip) in GPUI and in a web view at
  the same size, vary one of font weight, line height, size rounding and rasterization at a time,
  and record in this file which changes close the visible gap. Verify: before/after screenshots
  attached to the PR, with the chosen fixes listed.

  **Findings.** GPUI captured via the Metal renderer at 2x, against WKWebView at 2x, same strings,
  sizes, weights and colours; measured as ink coverage and mean absolute difference (MAD) from WebKit.

  | Variable | Effect |
  |---|---|
  | Font weight (variable font) | **The whole gap.** GPUI has no font-variation support and picks one face per family; `Manrope-Variable.ttf`'s default instance is wght 200, so every weight drew ExtraLight. Static instances bring MAD vs WebKit from 0.181 to 0.030 (WebKit's own smoothing on/off noise is 0.026). |
  | Line height (1.25 / 1.5 / 1.618) | None on glyphs; box height only. |
  | Size rounding (+0.25px) | None on sharpness. |
  | Fractional y (+0.25px) | None; GPUI snaps glyphs vertically. |
  | Rasterization / smoothing | Already matches WebKit (ink 3607 vs 3610). |

  **Fixes (2.3):** bundle Manrope as static per-weight instances (400/500/600/700, 800 if used)
  registered under family "Manrope" instead of the variable file, and never bundle a variable font
  for GPUI. No changes to line height, size rounding or rasterization. Screenshots:
  `assets/compare-before-after.png`, `assets/zoom-4x-before-after-webkit.png`.

## 2. Fonts

- [x] 2.1 Bundle Adamina (with its OFL text) next to Manrope and register it at startup, adding
  frame, data and code font roles to the theme. Verify: a test that all three families are
  registered and resolve to bundled files.
- [x] 2.2 Apply the frame role to panel titles, tabs, section headers, buttons, menus, the context
  bar, the status bar and dialogs, and the data role to table cells, field values, chips and cards.
  Verify: render tests asserting the family on a panel title, a tab, a table cell and a chip.
- [x] 2.3 Apply the rendering fixes found in 1.1. Verify: the tests or screenshots named in 1.1.

## 3. Spacing tokens

- [x] 3.1 Add `ui::space` with `panel_inset`, `card_padding`, `row_height`, `section_gap` and
  `control_gap`, roomier than today. Verify: unit tests for the scale's values and their scaling.
- [x] 3.2 Convert panels, cards, tables and dialogs to the tokens in groups (detail panels; tables;
  dialogs and windows; context and status bars). Verify: per group, a render test that content is
  inset from the panel edge by at least `panel_inset`.

## 4. Text size

- [ ] 4.1 Add the text-size preference (scale factor, bounded, default 100%), persisted in the
  preference file, applied to every open window live and to the spacing tokens. Verify: tests for
  bounds, persistence round-trip, and that a size change re-renders an open window larger.
- [ ] 4.2 Add Increase / Decrease / Reset Text Size commands (⌘= / ⌘- / ⌘⇧0) in the View menu and
  palette, and a keyboard-operable stepper in Settings. Verify: `simulate_keystrokes` tests for
  each command and for the Settings stepper.

## 5. Verification

- [ ] 5.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, `cargo test`.
- [ ] 5.2 Manual check: compare a few panels before and after side by side; confirm frame text is
  Adamina and data is Manrope, spacing is roomier and even, text reads smoothly, and text size
  changes by command and in Settings and survives a relaunch.
