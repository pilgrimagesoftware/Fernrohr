# Design

## Context

`k8s/resource/events.rs::list` does a one-shot field-selected list (`involvedObject.*`) when the
pod detail panel loads; `summarize` sorts by `event_time` (which already handles `events.k8s.io/v1`
fields). See proposal.md for why.

## Goals / Non-Goals

**Goals:** live events per pod detail panel; a client-side time window; a persisted default.

**Non-Goals:** the object-detail panel's events (same mechanism can be adopted later); server-side
time filtering (the API has no time field selector); events for the pod's owners.

## Decisions

### D1: A per-panel field-selected watch
Replace the one-shot list with a `kube_runtime::watcher` on `Event`, using the existing
`involvedObject` field selector, started when the panel opens and dropped with it. Not shared via the
watch registry: the selector is unique per pod, so there is nothing to share. Reuse
`watch_stream::run` for backoff, 401 and 403 handling; a 403 becomes the existing "could not be
listed" state.

### D2: The window is a client-side filter on `event_time`
The API can't filter by time, so the panel keeps every watched event and filters on `event_time`
against the window when rendering, re-evaluating once a minute so events age out. Hidden count =
retained minus shown.

### D3: Preference
`pod_events_window` in `ui.toml` (default 1 hour). A panel starts from it; changing the selector
updates the panel and writes the preference. Commands "Pod Detail: Events Window - <n>" in the
palette, panel-scoped keys on the Events tab.

## Risks / Trade-offs

- [Event watches per open pod panel] -> field-selected to one object, so each is cheap; torn down
  with the panel.
- [Clusters retain events ~1h by default] -> "All" and long windows can't show more than the cluster
  keeps; the tab's hidden/empty wording says "no events in window", not "no events ever".
