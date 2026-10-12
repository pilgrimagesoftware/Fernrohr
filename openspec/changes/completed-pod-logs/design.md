# Design

## Root cause: the line list is replaced, not appended to

The app already fetches the output. `stream.rs::log_params` asks for `follow: true` on the current
instance, and the Kubernetes API answers a follow request for an already-terminated container by
sending its whole log and then closing the stream. `stream.rs::produce_container_logs` forwards
each line as `LogEvent::Line` into `LogsView::append_line`, and when the stream closes it sends
`LogEvent::Ended`. So for a completed Job's pod, `LogsView.lines` holds the full output.

What is lost is the display. `LogsView::apply` handles `LogEvent::Ended` by setting
`terminal_message = Some("Log stream ended.")`, and `render.rs::LogsPanel::render` begins its
content choice with `if let Some(message) = view.terminal_message()` and renders
`panel_title::error_content(message, detail, cx)` - the same full-panel error view used for a
failed request - *instead of* `self.lines_list(view.lines(), cx)`. A stream that has a terminal
message therefore never shows its lines. For a long-running pod this goes unnoticed until the pod
dies; for a pod that has already completed, the stream is read and ended within milliseconds of
opening, so the user only ever sees the end notice. This contradicts the existing "Pod terminates
while streaming" scenario ("retains the lines already shown").

Ruled out, from the code:
- `tail_lines` / `sinceSeconds`: `restart` passes `tail_lines: None` and no `since_seconds`; the
  tail limit is only set by label-following panels (`labels/follow.rs`).
- Error swallowed: failures to start map to `RequestFailed` and show a real reason; a read error
  mid-stream does too. A clean close is the only path to the bare "ended" text.
- Wrong container: `restart` streams the container the panel selected.
- Succeeded phase: no phase handling exists; the pod phase is never consulted.
- Multi-container tagged streams (`apply_from`) already append the end as a line after the output,
  so they do not have this bug.

## Ended is a marker after the content, not a replacement for it

`LogsView::apply` splits the two terminal cases by kind rather than by shared field. A plain end
gets its own state (`ended: bool`, or the existing `terminal_message` kept for failures only) so
`render` can tell them apart:

- `RequestFailed` and `NoPreviousInstance` keep the full-panel `error_content` view - there are no
  lines to show and the message is the whole story.
- `Ended` keeps `lines_list(view.lines(), ..)` and appends a muted "log stream ended" row below the
  last line (the same wording `apply_from` already writes for tagged streams). With zero lines it
  reads "The container wrote no log output." so an empty log is distinguishable from a failure.

The ended row is part of the list's content, not a separate widget under it, so scrolling and the
follow/jump math keep working; it is rendered as one extra item after `lines.len()`, and
`line_count` for scroll purposes counts it. The control bar's `streaming` flag
(`terminal_message().is_none()`) must stay false after an end, as it does today.

## Non-follow for terminated containers

A follow request on a terminated container works but relies on the server closing the stream.
Reading to the end with `follow: false` is the honest request: "give me what exists." `LogTarget`
gains `follow: bool`, and `log_params` becomes `follow: target.follow && !target.previous`, so the
previous-instance path (already non-follow) is unchanged. `LogsPanel::restart` sets it from the pod
data it already has: a container whose status is `terminated` (or a pod in phase `Succeeded` or
`Failed`) is non-follow. A container that is running, waiting, or has no status yet stays follow,
so a restart-looping pod still streams. If the status is missing, follow is the default - that is
today's behavior and degrades safely.

Follow-mode UI for a non-follow stream: the Follow toggle has nothing to follow once the content is
all there, so the view starts at the top-of-output behavior it has today (`Following` pins to the
bottom, which for a finished log shows the end and the marker). No new control is added.

## `previous=true`: out of scope

The Previous toggle (`toggle_previous`, `LogEvent::NoPreviousInstance`) already exists and works
for a restarted container; it is a different feature (a prior instance of a still-running
container). A completed Job's pod has no previous instance - its only instance is the terminated
one this change fixes. Defaulting to previous for a restarted pod that is now terminated, or
auto-falling-back when the current instance has no output, is a real follow-up but not required to
answer #227, and is noted rather than bundled.

## Pod garbage collection

If the Job's pod has been deleted (TTL, cleanup), the API returns 404 and the existing
`describe_log_stream_error` message "Pod ... not found" applies. Nothing in this change can show
logs for a pod that no longer exists.
