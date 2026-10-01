# Tasks

## 1. Build stamping

- [x] 1.1 Add `App/app/build.rs` stamping `FERNROHR_BUILD_COMMIT` (short hash, `-dirty` suffix on
      an unclean tree, `"unknown"` sentinel when `git` or the repo is unavailable) and
      `FERNROHR_BUILD_DATE` (UTC `YYYY-MM-DD`) via `cargo:rustc-env`, with
      `cargo:rerun-if-changed` on `.git/HEAD`; verify `cargo build -p app` succeeds and
      `env!("FERNROHR_BUILD_COMMIT")` resolves in a throwaway `cargo expand`/compile check.

## 2. `build_info` module

- [x] 2.1 Add `app/src/ui/about_window/build_info.rs` with `version()`, `build_identifier()`, and
      `build_details()` functions mirroring Knot's (`CARGO_PKG_VERSION`, date+commit, and the
      combined bug-report line), matching on the `"unknown"` sentinel to show an explicit
      "commit unknown" string instead of the raw token.
- [x] 2.2 Add unit tests covering: version matches `CARGO_PKG_VERSION`; a known commit formats as
      `date, commit`; an unknown commit says so and never leaks the raw sentinel; the copyable
      details line contains app name, version, and build identifier. Verify with
      `cargo test -p app build_info`.

## 3. Window and pane

- [x] 3.1 Add `app/src/ui/about_window/window.rs`: the About window entity, a
      `register_about_action`-equivalent that owns an `Rc<RefCell<Option<AnyWindowHandle>>>` and
      opens-or-raises on the `About` action, and an `about_window_options`-equivalent
      `WindowOptions` builder (fixed size, no title on macOS, titled elsewhere, not resizable/
      minimizable). Verify by opening the window manually (`cargo run -p app`, trigger About
      twice) and confirming only one window exists.
- [x] 3.2 Add `app/src/ui/about_window/pane.rs`: the `Render` impl showing the
      `images/fernrohr-icon.png` icon (128px), app name, version/build text in a clickable row
      that copies `build_details()` to the clipboard via a tooltip-hinted control, a copyright/
      credits block naming GPUI and the author, and an `Escape` key handler that closes the
      window. Add the Close button variant for non-macOS via `cfg!(target_os = "macos")`.
- [x] 3.3 Add `app/src/ui/about_window/mod.rs` wiring the three submodules and exporting the
      action-registration entry point, matching Knot's `mod.rs` doc-comment split.

## 4. Wire into the app

- [x] 4.1 Remove `about_window`/`AboutView` from `app/src/ui/menu.rs`; call the new module's
      registration function from `init` (or wherever startup wiring lives) and have the `About`
      action's handler delegate to it. Verify `cargo build -p app` succeeds with no dead code
      warnings for the removed items.
- [x] 4.2 Update/extend `app/src/ui/menu.rs`'s existing About-related tests (the ones asserting
      the action dispatches and the menu item's position) so they still pass against the new
      wiring. Verify with `cargo test -p app ui::menu`.

## 5. Manual verification

- [ ] 5.1 Run the app, open About from the menu and from the command palette, confirm icon/name/
      version/build render, click the version/build text and paste the clipboard contents
      somewhere to confirm the copied line, press Escape to confirm it closes, and reopen to
      confirm a fresh instance opens after close. Record the result in this change's notes.
