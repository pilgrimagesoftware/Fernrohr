# Design

## Context

See proposal.md - Why. The reference implementation is Knot's own About window
(`pilgrimagesoftware/Knot` App repo, `crates/knot/src/about_window/`), which splits into
`build_info` (which binary is running - testable, no window), `window` (single-instance open/
raise + the entity), and `pane` (what it draws). Fernrohr has no `build.rs` yet and no
single-instance window pattern to reuse; this change introduces both, scoped to the About window.

Fernrohr has no i18n infrastructure wired up today (`rust-i18n` isn't a dependency, and existing
UI strings - menu items, settings labels - are plain Rust string literals throughout). Knot's
`knot_core::l10n::t()` calls aren't portable to Fernrohr without first building that
infrastructure, which is out of scope here.

## Goals / Non-Goals

**Goals:**
- Match Knot's About window's observable behavior (single instance, icon, version + build,
  copyable details, Escape-to-close) using Fernrohr's own crate layout and `gpui_kit` surface.
- Keep `build_info`-equivalent logic unit-testable without opening a window.

**Non-Goals:**
- No i18n infrastructure. Strings are plain literals, consistent with the rest of the app's
  current UI code. Revisit if/when the app adopts `rust-i18n`.
- No generalized single-instance-window helper. The handle-holding closure pattern is written
  once for About; extracting it for reuse (e.g. a future Settings window) is deferred until a
  second window needs it.
- No credits beyond naming the toolkit (GPUI) and author - no derivation note like Knot's
  (Fernrohr isn't derived from another app).

## Decisions

- **Module layout**: `app/src/ui/about_window/{mod.rs, window.rs, pane.rs, build_info.rs}`,
  mirroring Knot's split. `window.rs` owns the `Rc<RefCell<Option<AnyWindowHandle>>>` and the
  `register_about_action`-equivalent wiring; `pane.rs` is the `Render` impl; `build_info.rs` is
  pure functions over `env!()` values, independently unit-tested.
  - Alternative considered: keep everything in one file as today. Rejected because `build_info`'s
    value (testable without GPUI) is lost if it's inlined into the `Render` impl.
- **Build stamping**: new `app/build.rs`, modeled on Knot's - `git rev-parse --short=12 HEAD`
  plus a `-dirty` suffix from `git status --porcelain`, falling back to an `"unknown"` sentinel
  the About window matches on explicitly (never shows the raw sentinel to the user). Build date
  via `time::OffsetDateTime::now_utc()` if the `time` crate is already a dependency, otherwise
  `std::time::SystemTime` formatted by hand to avoid a new dependency for three integers.
  - Env var names: `FERNROHR_BUILD_COMMIT` / `FERNROHR_BUILD_DATE` (Fernrohr's own prefix, not
    Knot's `KNOT_*`).
  - `cargo:rerun-if-changed=../.git/HEAD` (and the ref it points at) so a new commit re-stamps the
    binary without a `cargo clean`.
- **Icon source**: reuse `images/fernrohr-icon.png` (already shipped for packaging) via
  `include_bytes!`, rendered at 128px edge length as Knot does for its macOS-sized About icon.
- **Window chrome**: no titlebar text on macOS (the body already names the app); a titled window
  elsewhere, matching Knot's platform split and Fernrohr's own `about_window_options`-equivalent
  helper added alongside the existing ad hoc `WindowOptions` construction in `menu.rs`.
- **Wiring into `menu.rs`**: `menu.rs` keeps the `About` action and its registry entry; its
  `on_action` handler calls the new module's `open_about_window` (or equivalent) instead of
  constructing the window inline. The single-instance handle is created once in `init`, matching
  how Knot's `register_about_action` is called from its own startup path.

## Risks / Trade-offs

- [`git` absent on a packaging/build machine] → `build.rs` must not fail the build; falls back to
  the `"unknown"` sentinel, same as Knot's `commit() -> Option<String>` returning `None`.
- [Clipboard access differs per platform backend in GPUI] → already exercised elsewhere in the
  app (if any existing copy-to-clipboard action exists) or, if this is the first, follows
  `gpui_kit`'s documented `App::write_to_clipboard` / `ClipboardItem` API Knot already uses
  successfully on both macOS and Linux.
- [Scope creep into a general single-instance-window helper] → explicitly a non-goal; keep the
  pattern local to this module until a second caller exists.
