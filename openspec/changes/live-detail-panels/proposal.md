# Proposal

## Why

A detail panel reads its object once, when it opens. A pod that starts Pending - waiting on a missing
Secret, say - stays shown as Pending after the Secret is created and the pod is Running, even though
the Pods table beside it has long since updated. A detail view that silently goes stale is worse than
none: the user acts on the wrong state.

## What Changes

- The pod detail panel follows its pod live: phase, conditions, container states, restarts and every
  other field update as the cluster changes, with no reopen or refresh.
- The object detail panel does the same for any kind.
- Both read from the shared per-(context, kind) watch the list panels already use, so a detail panel
  adds no API traffic when its kind's list is open, and starts that watch (reference-counted) when it
  isn't.
- A deleted object keeps the existing "survives its object's absence" behavior, now triggered live.

## Capabilities

### New Capabilities

(none)

### Modified Capabilities

- `pod-detail`: the panel's fields update live.
- `object-detail`: the panel's fields update live.

## Impact

- App: `pod_detail` and `object_detail` load paths subscribe to the session's Pods/kind watch
  registry instead of a one-shot `get`; the YAML view re-renders from the latest object.
