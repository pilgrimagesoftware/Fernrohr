# Proposal

## Why

The Logs panel's Follow button (Fernrohr-App #204) gives little sign of whether follow is on or
off: it is a ghost `xsmall` button whose `.toggled(following)` state is a faint tint that is easy
to miss against the neighbouring jump buttons. And turning Follow on does not reliably take the
user to the newest line - the click flips the view's follow state, but the list only scrolls when
the next line arrives, so a user who re-enables Follow after scrolling up is still looking at
old lines until output happens to resume. Both make Follow feel broken: the user cannot tell what
it is doing, and the one moment they ask for the tail, they do not get it.

## What Changes

- The Follow control shows its state unmistakably: on is a clearly filled/accented button, off is
  plain, and the tooltip names the state ("Following new lines - click to stop" /
  "Follow new lines as they arrive").
- Turning Follow on scrolls the list to the newest line right then, if it is not already there,
  rather than waiting for the next line to arrive.
- Turning Follow off, and scroll-wheel pause/resume, keep working as they do today; the visual
  state follows them, so scrolling up visibly flips the control to off.

## Capabilities

### Modified Capabilities
- `pod-logs`: the "Follow mode" requirement gains a visible on/off indication and an immediate
  scroll to the newest line when follow is turned on.

## Impact

- `app/src/util/logs/render.rs`: `stream_controls` - the `logs-follow` button's styling, tooltip,
  and click handler (which today only calls `LogsView::scroll_to_bottom` /
  `scroll_up` without moving the list).
- `app/src/util/logs/panel.rs`: `LogsPanel::follow` / `on_lines_scrolled` - a shared scroll-to-end
  helper so the Follow click, "Jump to bottom" and the resume-on-scroll path all use one place.
- `app/src/util/logs/view.rs`: `LogsView::follow_state`, `scroll_up`, `scroll_to_bottom` are the
  existing state; no change expected.
- Interaction with `logs-search-filter`: the scroll uses the same visible-line count that change
  introduces, so it lands on the last visible line while a filter is active.
