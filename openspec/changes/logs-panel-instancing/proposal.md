# Proposal

## Why

Opening a second pod's logs currently retargets the same Logs panel rather than opening a new
one - because there is structurally only one: `NavTarget::Logs` is a single, pod-agnostic target
per window, and `LogsPanel` reactively streams whichever pod is in the `SelectedPod` global. This
matches what a *reuse* default should do, but there is no way to get the other behavior - keeping
one pod's logs open while opening another's alongside it, the way the pod detail panel already
lets you (each pod's detail is its own panel, keyed by `NavTarget::Pod(PodRef)`). The user's ask:
**per-pod panels as the default**, with a preference and a one-shot override that give the reuse
behavior instead (pilgrimagesoftware/Fernrohr#167).

## What Changes

- A new `NavTarget::PodLogs(PodRef)` variant, mirroring `NavTarget::Pod(PodRef)`'s per-pod
  identity - opening it for a pod that already has its own logs panel open focuses that panel
  (the same dedup `PanelKey` already gives `NavTarget::Pod`), opening it for a different pod opens
  a new one alongside it.
- A `UiConfig` preference (`logs_panels = "per_pod" | "reuse"`, per pod by default) deciding the
  *default* when a user opens a pod's logs: a per-pod instance (`NavTarget::PodLogs`) or the
  single shared Logs panel (`NavTarget::Logs`, today's behavior). It is set in a Settings section
  and by palette commands.
- A modifier override at the point of opening logs (the `l` keybind, the Pods table's log action,
  any future "open logs" entry point) that does the *opposite* of the configured default for that
  one invocation - so either default stays a one-key override away from the other behavior
  without changing the preference.
- `NavTarget::Logs` (the singleton, reactive-to-`SelectedPod` target) stays exactly as it is today
  for the reuse case - this is additive, not a replacement of the existing panel's behavior.
- Another container of a pod whose panel is open switches that panel rather than opening a
  second, and a pod's panel saves and restores its pod and container.

## Capabilities

### Modified Capabilities
- `pod-logs`: adds the requirement that each pod's logs open in a panel of their own by default,
  with a reuse preference and a per-action override.

## Impact

- `app/src/ui/nav.rs`: new `NavTarget::PodLogs(PodRef)` variant, `PanelKey` dedup extends to it
  for free (same mechanism as `NavTarget::Pod`).
- `app/src/util/logs.rs`: `LogsPanel` needs to support two constructions - one reactive to
  `SelectedPod` (today's `NavTarget::Logs` panel), one pinned to a specific pod at construction
  (mirroring `PodDetailPanel::new`'s fixed-`PodRef` shape) and never reassigned by the global.
- `app/src/config/ui.rs`: new preference field on `UiConfig`; `app/src/ui/logs_panels.rs` the
  live preference; `app/src/ui/settings/panels.rs` its Settings section.
- Every call site that currently does `open_target(NavTarget::Logs, ...)` (the `l` keybind, the
  Pods table row's log action) needs to read the preference and the invocation's modifier state
  to decide which `NavTarget` variant to open.
