# Design

Option A/B/C (giving each tab its own visible close button and per-tab focus color) stays
undecided - see `proposal.md`'s decision note. This slice covers the `Cmd-W` keyboard route, which
is Option C's own fix for the close-button report's real complaint (the control being awkward to
reach), just not shaped as an inline per-tab button.

## Cmd-W routing

`Cmd-W`'s registered command keeps its single binding; only its handler's behavior changes:

1. If `cx.active_window()`'s `MainWindow` has a non-empty `DockPlacement::Center` dock, dispatch
   `ClosePanel` to it. This closes whichever panel is the active tab in that dock's focused
   `TabPanel` group - the same action the ellipsis menu's "Close" item already dispatches, so
   there's no new close behavior, just a new (keyboard) route to the existing one.
2. Otherwise, treat it as "close window": check whether any of the window's contexts has an
   active tunnel. If none, call `util::shell::close_window` directly, unchanged from today. If any
   context has one, open a confirm dialog (title "Close Window?", body naming which context(s)
   have an active tunnel) before calling `close_window` from the confirm button - mirroring
   `context_bar::open_disconnect_dialog`'s Cancel/Disconnect footer, with "Cancel"/"Close Window"
   instead.

A window with several disconnected (no active tunnel) contexts and an empty dock still closes with
no dialog - the confirmation is about not silently killing a live tunnel, not about the number of
contexts or panels that were open.

## Non-window targets

The About and Tunnels-manage windows aren't `MainWindow`s, so step 1's dock check is naturally
`None`/empty for them and they fall through to step 2's window-close path with no tunnels to
check - no special-casing needed beyond what `window_context_count`/`context_panel_count` already
do for non-`MainWindow` windows (return the picker-mode default rather than panicking).
