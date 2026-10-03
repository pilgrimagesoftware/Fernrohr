# Tasks

## 1. Decision

- [x] 1.1 Option C chosen (2026-09-30) - see `proposal.md`. Revisit Option A separately once the
  `gpui-component` 0.7.0+ changelog can actually be checked.

## 2. Cmd-W: close active tab, else close window

- [x] 2.1 Add a helper that reports whether the focused window's dock has an open panel (reuse
  `DockArea::is_empty(DockPlacement::Center, cx)`, already used by `watch_workspace`).
- [x] 2.2 Wire `Cmd-W`'s registered command so its handler dispatches `ClosePanel` when the dock
  has an open panel, and falls through to the window-close path otherwise.
- [x] 2.3 Add an "any tunnel active on this window's contexts" check (reuse/extend whatever
  `ClusterSession`/`TunnelStore` already tracks live tunnel state for a context).
- [x] 2.4 When that check is true, show a confirm dialog before closing (naming the context(s)/
  tunnel(s) that will disconnect), modeled on `context_bar::open_disconnect_dialog` /
  `context_lifecycle::disconnect_confirmation_body`. When false, close immediately - same as
  today's `CloseWindow` handler.
- [x] 2.5 (Partly done in `tab-keyboard-navigation`'s App PR: `Cmd-W` closes the focused tab, and
  the displayed tab from the Resource panel, keeping the window; with no tab on screen it closes
  the window. The tunnel decision and dialog text are unit-tested. A keystroke test of the confirm
  dialog itself needs a live forward, which the shell tests can't build yet.) Keyboard test: `Cmd-W` with a panel open closes only that panel (dock still open,
  window still open); `Cmd-W` with no panel open and a tunnel active shows the confirm dialog;
  confirming closes the window; `Cmd-W` with no tunnel active closes the window with no dialog.
  Closed 2026-10-02 with a known gap: Cmd-W behaviour shipped (tab-keyboard-navigation, then
  tab-close-buttons), and the tunnel decision and dialog text are unit-tested; a keystroke test of
  the confirm dialog itself still needs a live forward the shell tests can't build.

- [x] 2.6 Menu bar's "Close Window" item and its palette entry: confirm they route through the
  same handler so the confirmation isn't bypassable from the menu.
