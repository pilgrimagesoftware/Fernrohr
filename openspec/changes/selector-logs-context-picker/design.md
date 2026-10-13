# Design

## Context

See proposal.md - Why. The current code that shapes the approach:

- A typed label Logs panel is a `LogsPanel` whose `PanelScope` names one context. At construction
  it takes that context's `ClusterConnection` entity and observes it. On its first sync it
  subscribes to that context's shared Pods watch (`ClusterRegistry::subscribe_pods`), and it
  unsubscribes when the panel is released. Several detached `cx.observe` subscriptions are tied to
  the old context's entities.
- `MainWindow` files every open panel under a `PanelKey` (target, context, namespaces). That key is
  how "one typed panel per context" works: `on_action_show_label_logs` looks the panel up by it.
  `rescope` already re-files a panel whose namespaces change.
- Saved panel state already records `context_name`. A saved panel whose context the window doesn't
  hold is restored as a placeholder naming that context (`saved_layouts::load`, section 5).
- The Resource panel's header has the pattern to copy: when the window holds more than one context,
  a dropdown of checked context items. Picking one emits `ResourceEvent::SwitchContext`, so
  `MainWindow` stays the one place that writes window state.
- The typed panel's bar already has a namespace picker (`labels/scope.rs`) opened by `n`
  (`PICK_NAMESPACES_KEY`). The Logs panel's keys are `c`, `p`, `s`, `/` and `n`.

## Goals / Non-Goals

**Goals:**
- Switching context reuses the code that opens a typed panel, instead of rebinding a live panel's
  connection, watch and observers in place.
- The window keeps one typed panel per context, enforced in one place.

**Non-Goals:**
- A context control on workload Logs panels, single-pod Logs panels or list panels.
- Searching several contexts at once from one panel.
- Changing which context the panel opens on; App#268 already made that the focused list's.

## Decisions

### D1: Switching replaces the panel instead of retargeting it in place

Choosing another context asks `MainWindow`, through a new `ScopeEvent`-style event from the panel,
to open a typed panel in that context with the old panel's selector and container filter, scoped
to all namespaces. The new panel goes in the old one's dock slot, and the old panel is then closed.
Closing it releases it, which ends its streams and unsubscribes it from the old context's Pods
watch through the cleanup that already exists.

*Alternative considered:* retarget the live `LogsPanel` by swapping its connection, re-observing,
and moving its Pods subscription. Rejected: every detached observer tied to the old context would
need tearing down and re-creating, and missing one would leave the old context re-syncing a panel
that no longer searches it. Replacing the panel uses the open and release paths that are already
tested.

### D2: `MainWindow` enforces one typed panel per context

`MainWindow` handles the switch request. If the target context already has a typed panel (found by
`PanelKey`, as `on_action_show_label_logs` does), `MainWindow` focuses that panel and leaves the
requesting panel alone. Otherwise it does D1. The panel never decides this itself, so the rule
lives where the keys live.

### D3: The control copies the Resource panel's dropdown

The control is a dropdown of the window's contexts, with the current one checked, in the selector
bar beside the namespace picker. It shows only when the window holds more than one context, the
same rule as the Resource panel (`shows_cluster_dropdown`). Like that header, the dropdown only
emits the request; the panel doesn't write window state. The panel learns the window's contexts as
the Resource panel does, from the context list `MainWindow` pushes to its children
(`sync_context_children`), so the control appears or disappears as contexts are added or closed.

### D4: Keyboard route: a panel-scoped command, default key `x`

A registered command, "Logs: Pick Context…" (`logs.pick_context`), scoped to the Logs panel's
`KeyContext`, opens the dropdown with its current item selected. ↑/↓ move, Enter picks and Escape
closes. Its default key is `x`, which is free in the panel's context (`n` is already the namespace
picker's). The hint row shows the key from the live keymap while the control is visible. In a
window with one context the command does nothing, and the palette still lists it.

### D5: Saved layouts need no new state

The replacement panel (D1) is a new typed panel built in the new context, so it saves
`context_name` like any other panel. Loading a layout uses the existing path, and the existing
missing-context placeholder covers the last spec scenario. No layout format change is needed.

## Risks / Trade-offs

- [Replacement loses the old panel's log lines] → Intended: lines from the old cluster are
  unrelated to the new one, and the panel already starts a fresh view on a new selector.
- [The new panel lands in a different dock position than the old one] → D1 puts it in the old
  panel's slot before closing the old one. A test checks the tab position.
- [`x` collides with a future binding in the panel] → It's rebindable in `keymap.toml`, and the
  keymap tests flag a duplicate default within one `KeyContext`.
- [`selector-logs` isn't in `openspec/specs/` yet] → The delta only adds requirements, and the
  change is archived after `selector-logs-panel` (proposal.md).
