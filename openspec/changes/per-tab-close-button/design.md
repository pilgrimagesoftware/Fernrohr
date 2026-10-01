# Design

Option A/B/C (giving each tab its own visible close button and per-tab focus color) stays
undecided - see `proposal.md`'s decision note. This slice covers the `Cmd-W` keyboard route, which
is Option C's own fix for the close-button report's real complaint (the control being awkward to
reach), just not shaped as an inline per-tab button.

## Cmd-W routing (as shipped, `app/src/util/shell/tabs.rs`)

`Cmd-W`'s registered command keeps its single binding; only its handler's behavior changes, via
`MainWindow::on_action_close_window` (listened for on the window's root element, not the app-wide
menu handler - the only way a main window's `Cmd-W` can differ from Tunnels'/About's, which have
no tabs):

1. If the dock has a focused tab group with an active panel, focus that panel's own focus handle
   first, then dispatch `ClosePanel`. Focusing first (not just dispatching) is what lets `Cmd-W`
   close the on-screen tab even when focus is actually elsewhere in the window (e.g. the Resource
   panel) - `ClosePanel` only reaches a tab group along the focus path, same as the ellipsis
   menu's "Close" item.
2. Otherwise, treat it as "close window": `tabs::losing_a_tunnel` narrows the window's contexts to
   the ones both bound to a live forward *and* held by no other window - closing the window is
   what would actually drop those tunnels, not merely contexts that happen to have one. Empty ->
   `close_window` directly, unchanged from before. Non-empty -> a confirm dialog (title "Close
   Window?", body naming those contexts) before calling `close_window` from the confirm button -
   mirroring `context_bar::open_disconnect_dialog`'s Cancel/Disconnect footer, with "Cancel"/"Close
   Window" instead.

A context another window still holds keeps its tunnel regardless of this window closing, so it
never appears in the dialog - the confirmation is about tunnels this close would actually end, not
about how many contexts or panels were open.

## Non-window targets

The About and Tunnels-manage windows have no tab commands registered (`with_tab_actions` is
`MainWindow`-only), so `Cmd-W` there falls through to the app-wide `CloseWindow` handler instead -
no special-casing needed.
