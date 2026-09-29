# Proposal

## Why

Opening a second pod's logs currently retargets the same Logs panel rather than opening a new
one - because there is structurally only one: `NavTarget::Logs` is a single, pod-agnostic target
per window, and `LogsPanel` reactively streams whichever pod is in the `SelectedPod` global. This
matches what a *reuse* default should do, but there is no way to get the other behavior - keeping
one pod's logs open while opening another's alongside it, the way the pod detail panel already
lets you (each pod's detail is its own panel, keyed by `NavTarget::Pod(PodRef)`). Paul's ask:
default to reuse (current behavior, no regression), with an explicit override to open a new
instance instead.

## What Changes

- A new `NavTarget::PodLogs(PodRef)` variant, mirroring `NavTarget::Pod(PodRef)`'s per-pod
  identity - opening it for a pod that already has its own logs panel open focuses that panel
  (the same dedup `PanelKey` already gives `NavTarget::Pod`), opening it for a different pod opens
  a new one alongside it.
- A `UiConfig` preference (`logs_panel_reuse` or similar - see design.md) deciding the *default*
  when a user opens a pod's logs: reuse the single existing Logs panel (today's behavior,
  `NavTarget::Logs`) or always open a per-pod instance (`NavTarget::PodLogs`).
- A modifier override at the point of opening logs (the `l` keybind, the Pods table's log action,
  any future "open logs" entry point) that does the *opposite* of the configured default for that
  one invocation - so either default stays a one-key override away from the other behavior
  without changing the preference.
- `NavTarget::Logs` (the singleton, reactive-to-`SelectedPod` target) stays exactly as it is today
  for the reuse case - this is additive, not a replacement of the existing panel's behavior.

## Capabilities

### Modified Capabilities
- `pod-logs`: adds the requirement that a user can open more than one pod's logs simultaneously,
  with a configurable default and a per-action override.

## Impact

- `app/src/ui/nav.rs`: new `NavTarget::PodLogs(PodRef)` variant, `PanelKey` dedup extends to it
  for free (same mechanism as `NavTarget::Pod`).
- `app/src/util/logs.rs`: `LogsPanel` needs to support two constructions - one reactive to
  `SelectedPod` (today's `NavTarget::Logs` panel), one pinned to a specific pod at construction
  (mirroring `PodDetailPanel::new`'s fixed-`PodRef` shape) and never reassigned by the global.
- `app/src/config/ui.rs`: new preference field on `UiConfig`.
- Every call site that currently does `open_target(NavTarget::Logs, ...)` (the `l` keybind, the
  Pods table row's log action) needs to read the preference and the invocation's modifier state
  to decide which `NavTarget` variant to open.
