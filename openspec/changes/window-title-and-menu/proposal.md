# Proposal

## Why

Every Fernrohr window is anonymous. `open_window` passes `WindowOptions::default()`, whose
`titlebar` is `None`, so the native macOS title bar hides its text
(`setTitleVisibility_(NSWindowTitleHidden)`) and no window ever calls `set_window_title`. The
Window menu is installed as the real `NSWindowsMenu` (`ui/menu.rs` names it "Window", which
`gpui-pre-macos`'s `create_menu_bar` hands to `setWindowsMenu_`), so AppKit populates it from
each window's `NSWindow` title - and with those titles empty, two Fernrohr windows read as two
identical blank rows. A multi-cluster, multi-window app whose stated differentiator is running
several clusters side by side gives the user no way to tell which window is which, from the title
bar, the Window menu, Mission Control, or the accessibility tree (`set_window_title` also feeds
`a11y.set_window_title`).

## What Changes

- Each main window names its own state in its title, updated as that state changes rather than
  set once at open: the active cluster context in `Workspace` mode (e.g. `staging - Fernrohr`),
  the app name alone while it is still in the cluster picker, and a count of connected contexts
  when it holds several. Adding or disconnecting a context re-titles the window; disconnecting the
  last one puts it back to the picker title.
- The title is set through GPUI's own `Window::set_window_title`, so the same string reaches the
  native title bar, the macOS Window menu, and accessibility, and nothing is drawn twice in a
  custom title bar.
- The title is computed by one pure function from a window's mode, so the title bar, the Window
  menu and any future surface read one answer.

Not in scope: renaming or reordering windows, per-window user-set titles, native macOS tab groups
(`WindowOptions::tabbing_identifier`), and titles for the auxiliary About, Settings and Tunnels
windows - those are single-instance and already name themselves in their content.

## Capabilities

### New Capabilities

None.

### Modified Capabilities

- `app-shell`: a window's title names the cluster contexts it holds, and updates as they change.
- `application-menu`: the Window menu lists every open window by its own title, so two windows on
  different clusters are told apart there.

## Impact

- App code:
  - `util/shell/window.rs`: `open_window` passes a `TitlebarOptions` carrying the initial title
    instead of leaving `titlebar` unset.
  - A new `util/shell/window_title.rs`: the pure `title_for(mode)` function and its unit tests, plus
    the one `apply` that pushes a window's title at the OS. Kept out of `window.rs`, which is
    already near the file-size cap.
  - `util/shell/main_window.rs` / `contexts.rs`: `enter_workspace`, `enter_picker`, `add_context`
    and `disconnect_context` re-title the window. `sync_context_children` is the natural hook - it
    is already the single place every edit to `contexts` passes through - so a new context cannot
    appear without the title following it.
  - No new dependencies: `Window::set_window_title` is already in the build through `gpui-kit`.
- Tests: `#[gpui_kit::test]` coverage of `title_for` for every mode, plus a `TestAppContext` test
  that a window entering a workspace is titled with its context. No live cluster.
- Platform: the macOS behaviour depends on AppKit's `setWindowsMenu_` window list, which only
  appears once a window has a title; Linux picks the title up as the WM's `_NET_WM_NAME` /
  xdg-toplevel title. Both come free from the same call.
