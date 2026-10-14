# Design

## Context

See proposal.md - Why. The current code that shapes the approach:

- A typed label Logs panel's target is `NavTarget::LabelLogs(LabelLogs::Typed)`, a unit variant.
  `MainWindow` files every open panel under a `PanelKey` built from the panel's scope: target,
  context and namespaces. `open_target_in` focuses an open panel whose key matches instead of
  building another, so two typed panels in one context and scope would share a key and only one
  could exist.
- `on_action_show_label_logs` (App#268) looks up the open typed panel in the focused list's context
  and focuses it, otherwise it opens one there.
- `switch_label_logs_context` (selector-logs-context-picker D1/D2) replaces the panel with a new one
  in the chosen context, in its tab, unless that context already has a typed panel, which it
  focuses instead.
- A Logs panel saves `label_logs` as the serde form of `LabelLogs`, which is `"Typed"` today.
  `panels::panel_key` reads it to key a restored panel, and `register_restore` reads it again,
  separately, to build the panel.
- The typed panel's title is already `Logs: <selector>` once a selector is set, and `Logs: Labels`
  before that. `panel_title::title_element` adds the context in a window holding several.

## Goals / Non-Goals

**Goals:**
- Every typed panel has its own identity, so keys never collide however many are open.
- Saved layouts from before this change restore unchanged.

**Non-Goals:**
- Several panels for one workload. A workload's panel stays one per workload.
- A cap on how many typed panels can be open. Each panel's streams are already capped at 20.
- Renaming panels by hand.

## Decisions

### D1: `LabelLogs::Typed` carries an instance id

`LabelLogs::Typed` becomes `LabelLogs::Typed(TypedId)`, where `TypedId` is a newtype over `u64`. A
new panel gets a random non-zero id from `getrandom` (already a direct dependency), so ids need no
counter and can't collide with restored ones in practice. The id is part of the target, so it's
part of the `PanelKey`, and every typed panel is filed under its own key with no change to
`open_target_in`'s dedup rule.

*Alternative considered:* keep the unit variant and exempt typed panels from `open_target_in`'s
dedup. Rejected: the window would then hold several open panels under one key, and every lookup by
key (namespace sets, the focused-list lookup, restore pairing) would have to know which one it
meant.

### D2: Older saved state reads as `TypedId(0)`

`LabelLogs` deserializes either form, through a `serde(from = ...)` helper that is
`serde(untagged)`: the new `{"Typed": <id>}`, or the old bare `"Typed"`, which reads as
`TypedId(0)`. The id is fixed rather than random because `panel_key` and `register_restore` each
deserialize the same saved state. A random id would key the restored panel under one id and build
it with another. Older layouts held at most one typed panel per context, so `TypedId(0)` plus the
context is still unique. New ids are never 0.

### D3: Open Logs by Label… always opens a new panel

`on_action_show_label_logs` drops its existing-panel lookup. It opens
`LabelLogs::Typed(TypedId::new())` in the focused list's context and namespaces, else the active
context's, as App#268 decided. The new panel focuses its selector field on its first render, as an
empty typed panel already does.

### D4: Switching context keeps the replacement mechanism, without the focus branch

`switch_label_logs_context` keeps selector-logs-context-picker's D1: it opens a typed panel in the
chosen context in the switched panel's tab, carries over the selector and container filter, and
closes the old panel. That's what the user sees as the panel moving. The D2 branch, focusing the
chosen context's own typed panel, is removed. The replacement keeps the switched panel's
`TypedId`. The context is part of the key, so it doesn't collide with a typed panel that already
searches the chosen context.

## Risks / Trade-offs

- [An unbounded number of typed panels, each with up to 20 streams] → Each panel's cap is
  unchanged, and the user opens each panel deliberately. A window-wide cap can come later if it's
  needed.
- [Two empty panels both read `Logs: Labels`] → They're told apart once a selector is set. An empty
  panel is a moment's state and focuses its own field, so the panel the user is typing into is
  never in doubt.
- [Old layouts with `"Typed"`] → D2 reads them, and a restore test loads one.
