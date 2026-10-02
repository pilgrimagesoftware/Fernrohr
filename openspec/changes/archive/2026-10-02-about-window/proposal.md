# Proposal

## Why

`about_window` in `app/src/ui/menu.rs` is a stub: a fixed box with the app name and
`CARGO_PKG_VERSION`, no titlebar, no icon, no build identifier, and no way to get those details
out of the window short of retyping them. A user filing a bug cannot say which build they ran
beyond the released version number, and the window itself says so in its own doc comment
("the smallest honest thing that works"). Fernrohr already has a packaging icon
(`images/fernrohr-icon.png`) and no competing modal/dialog system, so there's no blocker left to
replace the stub with a real one.

## What Changes

- Replace the placeholder `about_window`/`AboutView` in `app/src/ui/menu.rs` with a dedicated
  `about_window` module: a single-instance window (reusing an already-open instance rather than
  stacking a second) showing the packaging icon, app name, version, and a build identifier
  (date + commit).
- Add a `build.rs` to the app crate that stamps the built commit (short hash, `-dirty` suffix on
  an unclean tree) and build date into the binary via `cargo:rustc-env`, falling back to an
  explicit "unknown" marker when there's no repository or `git` to ask (source tarball, missing
  `git`) rather than a blank field.
- Clicking the version/build text copies a single bug-report line (app name, version, build) to
  the clipboard.
- `Escape` closes the window while it's focused; on non-macOS platforms an explicit Close button
  is shown too, since there's no window-chrome convention to close it otherwise there.
- Credits block naming the toolkit/author, matching Fernrohr's actual stack (GPUI, not Knot's).
- **BREAKING**: none — this only changes the content of a window nothing else depends on.

## Capabilities

### New Capabilities

- `about-window`: the About window's single-instance behavior, content (icon, version, build
  identifier, copy-to-clipboard), and keyboard/close behavior.

### Modified Capabilities

(none — `application-menu` already requires an "About Fernrohr" item dispatching to this window;
that contract doesn't change, only what the window it opens contains)

## Impact

- `app/src/ui/menu.rs`: remove `about_window`/`AboutView`, call into the new module instead.
- New `app/src/ui/about_window/` (or similar) module: window/pane/build-info split, mirroring the
  separation of "what it draws" from "which binary is running" so the build-info half is testable
  without a window and reusable by a future `--version` flag.
- New `app/build.rs`: stamps `FERNROHR_BUILD_COMMIT` / `FERNROHR_BUILD_DATE` env vars at compile
  time; re-runs when `.git/HEAD` changes.
- No new dependencies: clipboard and image rendering already go through `gpui_kit`.
