# Design

## Context

See proposal.md for motivation. The mechanics this change rides on, all read out of the pinned
`gpui-pre` 0.3.6 / `gpui-kit` 0.6.6 sources:

- **`open_window` never sets a title.** `util/shell/window.rs` passes `..Default::default()`, and
  `WindowOptions::default()`'s `titlebar` is `None`. On macOS `titlebar.is_none_or(appears_transparent)`
  drives both `setTitlebarAppearsTransparent_(YES)` and
  `setTitleVisibility_(NSWindowTitleHidden)`, so the title bar draws no text at all. No
  `set_window_title` call exists anywhere in the app.
- **The Window menu is already the real thing.** `gpui-pre-macos`'s `create_menu_bar` special-cases
  `menu_config.name == "Window"` and calls `app.setWindowsMenu_(menu)`. AppKit then owns that menu's
  window-list section and builds it from the app's `NSWindow`s. Nothing about the window list has to
  be built by hand - but it can only be labelled, because `set_title` is the only place that both
  `setTitle:` and `changeWindowsItem:window title:filename:false` are sent, and nothing sends it.
- **`set_title` is three surfaces for one call.** `Window::set_window_title` forwards to
  `platform_window.set_title` (macOS: `NSWindow.title` + `changeWindowsItem`; Linux X11: `_NET_WM_NAME`;
  Linux Wayland: `xdg_toplevel.set_title`) *and* to `a11y.set_window_title`. The test platform stores
  it in `TestPlatformWindow::title`, so `window.window_title()` can be asserted in a
  `#[gpui_kit::test]` - the same way `about-window` and the menu tests already assert on real GPUI
  state.
- **Every edit to a window's contexts already funnels through one method.**
  `MainWindow::sync_context_children` (`util/shell/contexts.rs`) is documented as the one place that
  pushes `contexts`/`active` to the Resource panel, the status bar and the context bar;
  `add_context`, `disconnect_context` and `set_active_context` reach it; `enter_workspace` and
  `enter_picker` do not. The title can hang off the same hook, with those two titling the window
  themselves.

## Goals / Non-Goals

**Goals:**
- One pure function turns a window's mode into its title, so the native title bar, the Window menu
  and accessibility cannot disagree.
- One place pushes that title to the OS, called from the existing context-sync hook.
- Initial title at open time, so a restored window is never briefly blank.

**Non-Goals:**
- Drawing a custom in-window title bar. `gpui_kit::component::TitleBar` exists and would let the
  context name sit inside the app's own chrome, but it is a different visual design with its own
  drag and traffic-light handling, and it does not feed the Window menu. Rejected: the native title
  bar is what AppKit's window list reads.
- Retitling the About, Settings and Tunnels windows (proposal.md's non-goals).

## Decisions

### 1. The title comes from `Window::set_window_title`, not from a drawn element

One call reaches the native title bar, the Window menu's window list, the Wayland/X11 window title,
and the a11y tree. Drawing the name into the app's own chrome would satisfy only the first.

*Alternative:* render a `TitleBar` child holding the context name. Rejected - see Non-Goals.

### 2. `titlebar: None` becomes `Some(TitlebarOptions { title, .. })`

On macOS the title bar's text visibility is decided at window creation from
`titlebar.is_none_or(appears_transparent)`. Passing `Some(TitlebarOptions { title: Some(..),
appears_transparent: false, traffic_light_position: None })` both shows the text and sets the
initial title. `set_title` is then called again on every context change; the second call is
`changeWindowsItem:title:`, which is exactly what relabels the already-open Window-menu entry.

*Alternative:* leave `titlebar: None` and rely on the runtime `set_window_title` alone. Rejected -
it would set `NSWindow.title`, but `setTitleVisibility_(Hidden)` was already applied at creation and
is never revisited, so the title bar would still show nothing. The title would reach the Window
menu and a11y but not the bar.

### 3. `appears_transparent: false` and no traffic-light override, so the window keeps AppKit's own bar

The app has never drawn its own title bar, so the least surprising result is the standard macOS bar
with the window's name in it. `TitleBar::title_bar_options()`'s `appears_transparent: true` and
`traffic_light_position: Some(px(9.), px(9.))` are for the custom-titlebar path and would fight
`Root`/dock layout that has no such bar to make room for.

*Alternative:* adopt `TitleBar::window_options()` wholesale for a unified look. Deferred to a change
that wants the custom bar; it is a visual redesign, not a titling fix.

### 4. The title is a pure function of the mode, unit-tested without a window

`window_title::title_for(&WindowMode) -> String`:

- `WindowMode::Picker(_)` -> `"Fernrohr"`.
- `Workspace { contexts, active, .. }` with `contexts.len() == 1` -> `"<context> - Fernrohr"`.
- with `contexts.len() > 1` -> `"<n> clusters - Fernrohr"`.

The app name comes from `env!("CARGO_PKG_NAME")`'s product name, matching what About already shows,
so the title cannot drift from the About window. `active` indexes `contexts`, so a title names the
context whose panels the Resource panel is pointed at - the same one the dropdown shows.

*Alternative:* name every context joined by commas (`staging, production - Fernrohr`). Rejected: it
grows without bound (a window can hold many contexts) and macOS truncates menu entries with an
ellipsis, so the distinguishing part would be the part that gets cut. A count is stable and short.

### 5. Re-titling hangs off `sync_context_children`

`add_context`, `disconnect_context` (with contexts remaining) and `set_active_context` all call
it, so the title follows every one of them from a single hook. Two edits are still needed outside
it:

- `enter_workspace` sets `Workspace` mode without calling `sync_context_children`, so connecting
  from the picker titles the window itself, once the mode is set.

- `enter_picker` sets `Picker` mode directly and does not sync children (it has none), so it
  titles the window itself.
- The `sync_context_children` body bails early in `Picker` mode and when `contexts[active]` is
  missing, so the hook's title call sits after that guard.

*Alternative:* call the title setter from all four methods. Rejected - four call sites that can
drift is the exact failure `sync_context_children` exists to prevent.

### 6. `window.set_window_title` needs `&mut Window`, so the call is deferred off `Context<Self>`

`sync_context_children` takes `&mut self, cx: &mut Context<Self>` and has no `Window`. Its existing
body already wraps the child updates in `cx.defer`, for a different reason (avoiding re-entrant
`Entity::update` from a chip click). The title call joins that same `defer`, where a `WindowHandle`
update can borrow the window.

*Alternative:* thread `&mut Window` into `sync_context_children`. Rejected - it would change the
signature of the method `add_context`/`disconnect_context` call from inside their own subscriptions,
where the window is already mutably borrowed.

### 7. Tests read the title back off the window, not off the function alone

`window.window_title()` returns what `set_title` stored, so a `#[gpui_kit::test]` can open a window
through the real `open_window`, drive it into a workspace with the existing
`TestAppContext` helpers, and assert the title. That covers the wiring `title_for`'s unit tests
cannot. The Window-menu row itself is AppKit's, drawn from the same `NSWindow` title, so it needs no
separate test.

## Risks / Trade-offs

- **The native title bar is a visual change on every platform.** A window that showed no title text
  now shows one, and the macOS bar is no longer transparent. -> Intended; the app's own
  context bar still carries the per-context chips with health and tunnels, so nothing is lost, and
  the bar is a single fixed-height row the workspace column already reserves room for.
- **Linux is untested by hand here.** `set_title` reaches `_NET_WM_NAME` / `xdg_toplevel.set_title`,
  but a tiling WM may show the title only on hover. -> Accepted: the same call is correct on both,
  and the Window menu requirement is macOS-specific in its wording.
- **A long single context name can still overflow the title bar.** macOS truncates with an ellipsis
  and the full string stays in the Window menu and a11y name. -> Accepted; matching the full
  context name to the available width is a `title.rs` concern, not a window-title one.
- **`env!("CARGO_PKG_NAME")` is `fernrohr`, while the product name is `Fernrohr`.** Reading the
  wrong one puts `fernrohr - production` in the title bar. -> The task that builds the title
  constant uses the same source About already uses, and its unit test pins the exact expected
  string, so the casing is caught by `cargo test` rather than by eye.

## Migration Plan

None - no persisted state changes. Titles are derived from contexts the window already restores.

## Open Questions

- Whether the multi-context form should read "3 clusters - Fernrohr" or "3 contexts - Fernrohr".
  Settled here as "clusters" (the app's own word for a connection, per `connection-status-bar` and
  the picker); revisit only if a translation lands that reads badly.
