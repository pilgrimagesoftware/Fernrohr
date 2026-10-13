# Design

## Reuse what #150 landed; do not build a second panel

The typed-selector panel is `LogsPanel` with a `Following` state (`NavTarget::LabelLogs(Typed)`).
It reuses the single-pod panel's `LogsView` (follow state, scrollback), the virtualized
`uniform_list` renderer, and `start_tagged_stream`. This change adds state and controls to that
panel; it does not fork the renderer or the view.

## Opening empty

`ShowLabelLogs` (palette: "Logs: Follow Labels...") opens the panel with no selector and the
selector field focused. This change adds a menu item for the same action. One typed panel per
context: opening again focuses it, as today, so the user types another selector there rather than
collecting empty panels. Workload-opened panels (`LabelLogs::Workload`) are unchanged.

## Selector entry

A text field using `gpui_kit` `Input`, applied on Enter, parsed by `k8s::label_selector::parse`
(`kubectl -l` syntax: `k=v`, `k!=v`, `key`, `!key`, `k in (a,b)`, `k notin (a,b)`). Invalid text
shows the offending requirement and reason and keeps the previous selector streaming. An empty
selector, or one that selects everything, is rejected ("needs a requirement"): streaming every pod
in a cluster is never the intent, and the stream cap would make it arbitrary.

## Namespace scope

The panel already carries a `PanelScope` whose namespaces `label_namespaces()` reads. This change
surfaces that scope in the bar as a picker offering the current namespace(s), a named namespace set
(`namespace-sets`), or all namespaces. Choosing re-runs `sync_labels`. Empty namespaces means all,
matching `wanted`'s existing contract. No new storage: the scope already round-trips through the
saved layout's `namespaces`.

## Watching pods

No per-panel watch. The panel subscribes to the context's shared, reference-counted Pods watch
(`ClusterRegistry::subscribe_pods`, released on panel release) and calls `sync_labels` on every
change. `wanted` returns the started containers of matching pods; streams not wanted any more are
dropped, new ones started. A container waiting to start is picked up on its pod's next update. A
stream is keyed by namespace, pod, uid, container and restart count, so a recreated pod or
restarted container streams afresh while an unchanged one is never restarted.

## Streams, cap, tail

At most `LABEL_LOGS_MAX_STREAMS` (20) containers stream at once, like `kubectl logs -l`'s
`--max-log-requests`; the bar states "streaming 20" when more match. Selection under the cap is the
stable key order. Each stream starts at its last `LABEL_LOGS_TAIL_LINES` (100) lines so existing
history does not bury live output. Configurable cap and tail are follow-ups.

## Prefix and colour

Each line is tagged `pod` (or `pod/container` when that pod has several started containers).
Today the tag is concatenated into the line text. To colour it, `LogsView` keeps the source as a
separate field per line and the renderer draws it as a coloured span. Colour is a hash of the pod
name into a fixed palette drawn from theme-safe tones, so the same pod keeps its colour across
restarts, panels and re-syncs with no assignment state. Containers of one pod share the pod's
colour. Collisions are acceptable; the text prefix always disambiguates.

## Container filter

A picker in the bar: "All containers" (default) or one container name drawn from the matched pods'
containers. It narrows `wanted` (streams are stopped, not just hidden, so the cap is spent on
what is shown). With one container selected the prefix is the pod name alone. Selection is panel
state, saved with the layout.

## Interleaving order

Lines are appended in arrival order. Sorting by timestamp would need `timestamps=true` on every
request, a parse per line, and a merge buffer that delays live output to wait for slower streams -
the wrong trade for a tail. Within one container order is exact; across containers it is as exact
as delivery. Timestamp merge is a possible follow-up.

## Stream end and failure

A tagged stream's end or failure is a line in the panel (`LogsView::apply_from`), not the view's
terminal state, so other streams keep going. A pod that is deleted stops being wanted and its
stream is dropped.

## Persistence

The panel is saved in layouts: `label_logs` (source), `selector` (canonical text) and
`namespaces` already round-trip via `labels_from_state`. This change adds the container filter to
that state and restores it. A restored panel with a selector starts streaming on connect; with no
selector it opens empty.

## Deferred

Regex pod-name filter, exclude filters (`stern -e` / `--exclude-container`), `since` and tail
options, timestamp-ordered merge, configurable stream cap.
