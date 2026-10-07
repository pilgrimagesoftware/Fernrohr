# Proposal

## Why

Controllers that drive reconciliation - Flux's `HelmRelease` and `Kustomization` chief among
them - publish their last outcome as a human-readable message on their `Ready` condition. FreeLens
shows that message in both its resource list and object detail views (confirmed in design.md), so
a user can tell a stuck `HelmRelease` from a healthy one at a glance. Fernrohr has no equivalent:
these CRD kinds fall through `object_detail`'s kind dispatch to metadata-only panels and gain no
extra list column, so the only way to learn why a Flux resource isn't ready is to open its YAML and
read `status.conditions` by hand.

## What Changes

- Derive one toned status message for any object that reports `status.conditions`: the `Ready`
  condition's message when present, else the most recently transitioned condition that has a
  non-empty message. This is generic condition-reading, not Flux-specific - it applies to any kind,
  built-in or CRD.
- `resource-browser`: a list panel for a kind with no existing hardcoded column table (today, every
  CRD) gains a Message column once the panel observes that the kind's objects carry
  `status.conditions`. Kinds that already have a hardcoded column table (Deployment, Job, Node, ...)
  are unchanged - they keep their existing status-bearing columns rather than gaining a second,
  overlapping one.
- `object-detail`: the panel shows the derived message prominently, near the top, for any object
  that has one - both kinds that already render `status.conditions` as badges (Deployment,
  DaemonSet, StatefulSet) and kinds that today get metadata only. A kind with no hardcoded sections
  (CRDs, Flux kinds included) gains a generic Status section with the message and the full
  conditions list, in place of showing nothing beyond Overview.

## Capabilities

### New Capabilities

(none - this extends two existing capabilities)

### Modified Capabilities

- `resource-browser`: list panels gain a condition-derived Message column for kinds that report
  conditions and have no existing status-bearing column.
- `object-detail`: the panel surfaces a derived status message near the top for any object with
  conditions, and kinds with no dedicated sections gain a generic Status section (message +
  conditions) instead of metadata only.

## Impact

Rust-only change in the `App` submodule, no new external dependencies:

- `app/src/k8s/resource/status_tone.rs` - new generic condition-message derivation shared by both
  surfaces.
- `app/src/k8s/resource/object_list/columns.rs` - a new generic column path for kinds without a
  static `KindColumns` entry.
- `app/src/k8s/resource/object_detail/sections/{common.rs,mod.rs}` - a new generic Status section
  for the kind-dispatch fallback, and a shared Message field added to the kinds that already render
  Conditions badges.
- `app/src/k8s/resource/object_detail/model.rs` - no new `FieldValue` variant needed; reuses
  `Status` and `Badges`.

No backend, API, or persisted-state changes.
