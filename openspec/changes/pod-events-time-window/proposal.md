# Proposal

## Why

Pod events are the first place to look when a pod misbehaves, but in the pod detail panel they are
easy to miss: they sit behind the fifth tab, are listed once when the panel opens and never update,
and show everything the cluster still retains with no way to focus on what just happened.

## What Changes

- The Events tab updates live while the panel is open, instead of being a one-shot list.
- A time window selector on the Events tab (15 minutes, 1 hour, 6 hours, 24 hours, All) limits the
  list to events last seen within that window. The chosen window is the default for new panels and
  persists across restarts; each panel can change its own.
- The Overview tab shows the pod's recent Warning events within the window, so problems surface
  without switching tabs, with a link to the Events tab.
- The tab and Overview report how many events the window hides, so an empty list is never mistaken
  for "no events".

## Capabilities

### New Capabilities

(none)

### Modified Capabilities

- `pod-detail`: the Events tab becomes live and windowed; the Overview tab surfaces recent warnings.

## Impact

- App: `k8s/resource/events.rs` gains a watch alongside `list`; pod detail's Events and Overview
  tabs; a `pod_events_window` preference in `ui.toml`.
- One event watch per open pod detail panel, field-selected to that pod, torn down with the panel.
