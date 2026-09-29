# Design

## The widget: gpui-component's `Input`/`InputState`, not a hand-rolled text field

`gpui-component` (already a dependency) has `input::Input`/`input::InputState` - a real text-input
component (`Input::new(&Entity<InputState>)`). This app has never used it, but it exists and is
the library's own widget, so the pattern to establish here is "how this app wires `InputState`
into a panel," not "build a text field from scratch." `LogsPanel` gains an `Entity<InputState>`
field, constructed alongside `focus_handle`/`scroll_handle`, and observes it the same way it
observes `view` - a change re-filters and notifies.

## Filtering runs on the source list, not the rendered one

`LogsView::lines()` stays the full, unfiltered history (it's the thing tests already assert
against, and follow-mode/scroll-to-bottom math needs the real index space). Filtering happens in
`LogsPanel::render`, between reading `view.lines()` and handing a line count to `uniform_list`:
build the filtered `Vec<&str>` (or indices into `lines`) once per render, pass *that* length as
`item_count`, and index into the filtered set inside the row-render closure. This keeps
`LogsView` itself simple and keeps the filter a pure display concern, the same division
`sort_rows`/`view_rows` already draw in `pods.rs` between "what the data model holds" and "what
the current view shows."

One real interaction to get right: Jump-to-bottom/Follow-mode's scroll math (`line_count`,
`scroll_to_item`) has to use the *filtered* count while a filter is active, or "jump to bottom"
lands past the end of what's actually rendered.

## Substring only, no regex, no highlight - for now

Substring, case-insensitive, is the smallest thing that answers "find the line with X in it,"
which is the actual complaint. Regex support and highlighting the matched substring within each
line are both real, wanted features eventually, but each is its own scope (regex needs error
handling for an invalid pattern typed mid-edit; highlighting needs per-line rich text spans
instead of a plain string child) - bundling either into the first cut risks shipping neither
cleanly. Noted as explicit follow-up, not silently dropped.
