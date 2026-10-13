# Design

## The state already exists; it just is not legible

`LogsView::follow_state()` (`FollowState::Following` / `Paused`) is the single source of truth,
already driven by the Follow click, the wheel handler `on_lines_scrolled`, and the jump buttons.
`stream_controls` already passes `.toggled(following)` to the `logs-follow` button, so the data
path is right and the fix is presentation: pick a toggled style that reads at a glance (the
accent fill other toggled buttons in the app use, see `ui/accent.rs`) and make the tooltip state
the current mode. No new state is introduced, so the indicator cannot drift from the behaviour -
including when scrolling up pauses follow, which flips the button off on the next notify.

## Turning Follow on scrolls immediately, through one helper

Today the click calls `view.scroll_to_bottom()` and relies on `LogsPanel::follow`'s observer to
scroll when the view notifies; that scroll only runs if the view actually notified and the line
list is laid out, so the result is inconsistent. "Jump to bottom" instead scrolls explicitly with
`scroll_handle.scroll_to_item(line_count - 1, ScrollStrategy::Bottom)`. The Follow-on path should
do the same explicit scroll, and the two (plus the resume-on-scroll path in `on_lines_scrolled`)
should share a small `LogsPanel` helper so there is one definition of "go to the newest line".

"If not already there": `UniformListScrollHandle::is_scrolled_to_end()` (already used in
`on_lines_scrolled`) tells us; when it reports at-end, the scroll is skipped, which avoids a
redundant jump and any flicker. `None` (list not yet laid out) is treated as "scroll", matching
the existing `!= Some(false)` convention.

## Line count

The helper takes the line count of what is rendered. Without a filter that is
`view.lines().len()`; once `logs-search-filter` lands it is the filtered count, so Follow-on
never scrolls past the end of the visible list. This change does not depend on that one landing
first - it uses the count `render` already computes.
