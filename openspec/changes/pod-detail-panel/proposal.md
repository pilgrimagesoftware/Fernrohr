# Proposal

## Why

Selecting a pod in the Pods table currently renders its description or YAML inline, above the
table, inside the same `PodsPanel`. That means the list and the detail view fight for the same
docked space, can't be viewed side by side, and can't be moved, closed, or maximized
independently - all things every other panel in the app already gets for free from the dock.
Splitting them into separate panels (a "Pods" list panel and a "Pod" detail panel, opened per
selection) is the pattern every other resource kind in this app already follows for list vs.
single-item views.

## What Changes

- Opening a Pod's detail (via `DescribePod`/`ShowPodYaml`, or a new explicit "Open" action on a
  table row) opens a distinct dockable "Pod" panel, scoped to that specific pod, instead of
  rendering inline inside the Pods list panel.
- Selecting a Pod that already has an open detail panel for the same cluster focuses that panel
  rather than opening a duplicate, mirroring the existing resource-kind panel de-duplication rule.
- The Pods list panel keeps its table and per-row shortcuts hint but drops the inline
  description/YAML rendering entirely - detail is exclusively the new panel's job.
- The `ShowPodLogs` action continues to target the existing Logs panel/selection mechanism
  unchanged; this proposal only concerns the description/YAML detail views.

## Capabilities

### New Capabilities
- `pod-detail`: a dockable panel showing one Pod's description or YAML, opened from a Pods list
  selection.

### Modified Capabilities
(none - `resource-browser`'s requirements describe the list/table behavior, which is unchanged;
only where detail rendering lives is changing, and that is not a stated requirement there)

## Impact

- `app/src/k8s/resource/pods.rs`: `PodDetail`/`on_action_describe_pod`/`on_action_show_pod_yaml`
  move from `PodsPanel` state into a new panel type; `PodsPanel::render` drops the inline detail
  block.
- Dock/panel registration (`register_panel`, `PANEL_KEY_CONTEXT`-style key context) gains a new
  panel kind, following the existing `PlaceholderPanel`/`LogsPanel` pattern for panel-open and
  panel-focus-if-already-open.
- No config/persistence format changes; the new panel type needs its own `dump`/restore entry
  alongside the existing ones in saved dock layouts.
