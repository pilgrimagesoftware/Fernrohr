# Design

## Menu items are read from `CommandRegistry`, never duplicated

`CommandRegistry` (`app/src/command.rs`) already carries every command's `id`, `title`,
`default_binding`, and an optional `context` gate. The palette and keymap already read from it.
The menu bar becomes a third reader: build each menu's items by filtering `registry.iter()` for
commands tagged for that menu, rather than hand-writing a parallel list of titles/actions that can
drift from the palette.

This means `Command` needs one more field to answer "which top-level menu, if any, does this
command appear under": `pub menu: Option<MenuSlot>` where `MenuSlot` is a fixed enum mirroring the
seven top-level menus (`App`, `Context`, `Edit`, `View`, `Navigate`, `Window`, `Help`). `None`
means the command is palette/keymap-only (most commands, e.g. panel-scoped shortcuts) - the menu
is deliberately a curated subset, not every command dumped into seven buckets.

Menu item order within a menu follows registration order (the same order the palette lists them),
so both surfaces present commands consistently without a second ordering concept.

## The `Context` menu is Fernrohr's File menu, by design intent

macOS convention is a File menu. This app has no documents to open/save/close in the traditional
sense - what stands in that slot is choosing/switching which cluster context is connected. Rather
than force a "File" label onto content that isn't files, the proposal names it `Context` and gives
it the cluster-picker/switch-context actions. This is a deliberate naming choice to flag in review,
not an oversight.

## Static menu items that aren't commands

A few conventional items have no `CommandRegistry` entry and never will: `Quit`/`About` under
`App`, and standard `Window` items like `Minimize`/`Zoom` that GPUI's window chrome already
provides natively on macOS. These are constructed directly against GPUI's menu API
(`gpui::Menu`/`MenuItem::os_action` or equivalent) rather than routed through the command registry,
since they're platform affordances, not app actions - `on_action` dispatch would be redundant with
what the OS already does for them.

## Fonts: two `SharedString` fields on an existing `Theme`, no new plumbing

gpui-component's `Theme` (vendored dependency, not app code) already exposes `pub font_family:
SharedString` and `pub mono_font_family: SharedString`, defaulted to `".SystemUIFont"` and a
platform mono default respectively, and already read everywhere the app wants a font (e.g.
`cx.theme().mono_font_family.clone()` in the Pods YAML detail view and the Logs panel). Wiring
Manrope/Monaco is setting these two fields once, not introducing a new theme layer.

**Where this happens**: `crate::ui::theme::apply` runs after every `Theme::change`/
`sync_system_appearance` call (light/dark switch, system appearance change), so it's the single
place that already re-asserts theme state - font family assignment goes there, immediately after,
so a theme mode flip can't accidentally revert to the library's defaults.

**Font availability is the real risk, not the field**:
- Monaco is a macOS system font. It is not present on Linux, and this app explicitly targets
  Linux too (see `openspec/config.yaml`). Setting `mono_font_family = "Monaco"` unconditionally
  means Linux silently falls back to *some* substitute GPUI/the OS picks - acceptable for now
  (Linux is already the documented lower-maturity target), but the fallback chain needs to be a
  real font, not a typo'd name that renders nothing. Task list includes verifying the Linux
  fallback renders monospace text at all, not matching Monaco's look.
- Manrope is not a system font on macOS or Linux. It must ship as a bundled asset and be
  registered with `cx.text_system().add_fonts(...)` before `font_family` is set to it, or GPUI
  falls back to its own default and the setting is silently a no-op. Implemented as a plain
  `include_bytes!` of the TTF plus one `add_fonts` call - no `AssetSource` plumbing needed,
  `add_fonts` takes raw bytes directly, so the existing `gpui_kit::assets::Assets` source is
  untouched.

  **A real risk found while sourcing the file, not resolved by testing**: the only build of
  Manrope in the Google Fonts mirror is a single variable-weight TTF whose *legacy* name-table
  family (name ID 1) is `Manrope ExtraLight`, not `Manrope` - only its *typographic/preferred*
  family (name ID 16) is `Manrope`. Font libraries that don't handle variable fonts specially
  would see the family as `"Manrope ExtraLight"`, and setting `font_family = "Manrope"` against
  such a library would fail to match and (per gpui-component's own `mono_font.rs` doc comment)
  panic on first layout. Checked `zed-font-kit`'s macOS backend (`core_text.rs`) - it delegates
  family-name resolution to CoreText itself rather than parsing the name table by hand, and
  CoreText is documented to prefer the typographic family for variable fonts, so `"Manrope"`
  should resolve correctly on macOS, the primary target. This could not be exercised by a test
  (the test harness's font system reports no installed fonts at all) - flagged for the manual
  smoke test to confirm, and worth a second look if Linux's freetype/fontconfig path behaves
  differently.

## Non-goals restated from the proposal

No color/spacing/token work here - `mono_font_family`/`font_family` are the only two `Theme`
fields this change touches. Broader "design language" work stays unscoped until a future proposal
defines what it actually means.
