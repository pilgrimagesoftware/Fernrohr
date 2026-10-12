# Proposal

## Why

The user wants a Logs panel that opens empty, takes a set of labels, and streams every matching
pod's logs with the usual follow and scrollback - an arbitrary filter like the `stern` kubectl
plugin, not a panel tied to one pod.

Most of the mechanism already exists in the App (#150): `NavTarget::LabelLogs`, a `Typed` panel with
a selector field, a `kube` `Selector` parser (`k8s::label_selector`) accepting `kubectl -l` syntax,
streams started and stopped from the context's shared Pods watch, tagged lines, a stream cap and
a tail window, and layout restore. None of it has a spec, so it is unprotected and its gaps are
unnamed. This change specs the behaviour that is there and closes the gaps that make it a stern
equivalent rather than a bare label tail.

## What Changes

- Spec the existing typed-selector Logs panel as the `selector-logs` capability: opened empty,
  selector validated on Enter, matching pods streamed and followed as they appear and vanish,
  capped, restored from a saved layout.
- Per-line pod prefix gets a stable colour per pod (today the prefix is plain text), so
  interleaved pods can be told apart at a glance.
- A container filter in the panel's bar: all containers (the default) or one named container,
  applied across every matching pod.
- A visible namespace scope in the bar - the focused panel's current namespace(s) or namespace
  set (reusing `namespace-sets`), or all namespaces - editable without leaving the panel.
- An "Open Logs by Label" menu item beside the existing palette command, so the empty panel is
  discoverable without the palette.
- Lines are interleaved in arrival order, not timestamp order (see design.md); a stream that ends
  or fails is a line in the panel, not the panel's terminal state.
- Out of scope, listed as follow-ups: regex pod-name filter, exclude filters, `since`/tail options.

## Capabilities

### New Capabilities
- `selector-logs`: a Logs panel that follows every container a typed label selector picks.

## Impact

- `app/src/util/logs/labels.rs`, `labels/follow.rs`, `labels/render.rs`: container filter and
  namespace scope state in `Following`, colour assignment, the bar controls. `wanted` gains the
  container filter.
- `app/src/util/logs/view.rs`: the source tag a line carries is kept apart from its text so the
  renderer can colour the prefix.
- `app/src/util/logs/commands.rs`, `util/shell/logs_open.rs`: a menu entry for `ShowLabelLogs`.
- `logs-search-filter` (another pending change) touches the same panel: it adds a filter input
  to the control bar and filters `view.lines()` before `uniform_list`. The two compose - the
  substring filter runs over tagged lines, prefix included - but both edit `LogsPanel::render`'s
  bar and line-list construction, so whichever lands second rebases onto the other.
- No new dependency; reuses `gpui_kit` `Input`, `kube` `Selector`, and the shared Pods watch.
