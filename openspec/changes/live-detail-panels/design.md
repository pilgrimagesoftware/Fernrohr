# Design

## Context

`pod_detail/fetch.rs::fetch_pod` and the object detail fetch each do a one-shot `api.get`. The
session already runs shared, reference-counted watches per (context, kind) - `PodsTable` for Pods
and `ObjectsTable` per kind (`standard-resource-panels` D3) - including pause/resume, 401 refresh,
and the polling fallback for kinds that can't be watched.

## Decisions

### D1: Subscribe to the shared watch, look up by name (and uid)
A detail panel subscribes to its kind's shared watch for its context on open and unsubscribes on
close, then reads its object from that table by namespace and name, observing the table for changes.
*Alternative:* a field-selected watch per panel - simpler isolation, but a second stream of the same
data whenever the list is open, which is the common case.

### D2: First paint stays fast
If the shared table has not finished its initial list, the panel keeps today's one-shot `get` for its
first render, then switches to the table once it is synced. A uid mismatch (object deleted and
recreated under the same name) is treated as the old object's absence, then shows the new one.

### D3: View state survives updates
Tabs, scroll, revealed Secret values, expanded rows and folded YAML are keyed by field identity, not
by render, so an update re-renders values without resetting them.

## Risks / Trade-offs

- [A detail panel on a kind with no list open now holds a cluster-wide watch for that kind] ->
  reference-counted and dropped on close; for kinds that can't be watched it polls, as lists do.
