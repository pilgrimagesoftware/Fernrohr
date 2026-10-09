# Design

## Title: `LogsPanel` gets its own title text, not `panel_title::title`'s generic path

`panel_title::title`/`tab_name` derive purely from `PanelScope` (context/namespaces/target
label), which has no concept of "which pod, which container" - that identity lives on
`LogsPanel::current: Option<(String, String, String)>` (namespace, pod, container), set once a
pod's logs start streaming. Rather than stuff pod identity into `PanelScope` (which every other
panel type also carries and doesn't need this field), `LogsPanel::title`/`tab_name` build their
own string when `current` is `Some`: `"Logs: <pod> · <container>"`, falling back to the current
generic label when nothing is selected yet (the panel's existing "click a pod to view its logs"
empty state).

## Scroll: `gpui::uniform_list` instead of an unconditional `.children()` map

`gpui`'s own `uniform_list` element (`gpui_pre::elements::uniform_list`) renders only the visible
row range plus a small overscan buffer, recycling row elements as the list scrolls - this is the
standard fix for "N divs, one per data row, and N grows unboundedly" in this framework, and other
panels in this app already lean on gpui-component's `DataTable`/`TableState` for the same
underlying reason (the Pods table doesn't hand-roll a row list either). `LogsView::lines()`
already returns `&[String]` indexed by position, which is exactly what `uniform_list` needs (an
item count plus a per-visible-index render callback) - swapping the render call is a
render-function change, not a `LogsView` data-model change.

The existing `.whitespace_nowrap()` + `.overflow_scrollbar()` behavior (monospace, one line per
row, horizontal+vertical scroll) is preserved; only the mechanism producing the row elements
changes.

## Follow mode interacts with the new list, but isn't rebuilt here

`FollowState`/`scroll_to_bottom` are already `UNWIRED` (per their own doc comments - no scroll
handler calls them yet). Swapping to `uniform_list` doesn't fix that unrelated gap, but it does
mean whoever wires follow-mode next does it against a virtualized list's scroll-position API
rather than a plain `overflow_scrollbar` div's - worth a one-line note in that gap's doc comment
once this lands, not a task here.
