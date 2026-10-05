# Design

## Context

`bulk-select-list-actions` (in-flight alongside this change) adds a checked-set + bulk-action-bar
layer for actions that apply uniformly across one kind or a compatible group of kinds: delete,
restart/rollback rollout, scale, cordon/drain, label/annotate, view logs. This change is its
single-row counterpart for actions that only ever make sense for one specific kind and would be
nonsensical to generalize - you don't "scale" a ServiceAccount or "taint" a Service. These belong
on that one row's context menu and that kind's detail panel, not a bulk bar.

Port Forward for Pods and Services already shipped in `k9s-remaining-keybindings`: `shift-f` on a
Pods-table or Services row calls `K8sPortForwardConfig`/`PodPortForwardTransport`
(`App/app/src/forward/k8s/port_forward.rs`), resolving a Service to one of its ready endpoint Pods
first, and the resulting `ManagedForward` shows up in a "Port forwards" section of Manage Tunnels
with the same stop control an SSH tunnel has. That shipped work is reused unchanged here - this
change is only about the entry points it didn't add: a Pod detail panel action, a Service
object-detail section action, and a context-menu item on both rows, for a mouse user with no
reason to know the keybinding.

One more thing already exists that this change is the first to actually call:
- `kube-client`'s `create_subresource("token", ...)` helper - present in the dependency, unused
  by the app.

Every list row and detail panel today is read-only except for the actions
`bulk-select-list-actions` adds and the single-object Copy/reveal actions `object-detail`/
`pod-detail` already have, plus the row-level keyboard/palette commands
`k9s-remaining-keybindings` shipped (delete, kill, edit, shell, port-forward, previous logs).
`ObjectListPanel`'s row right-click menu exists today but only offers "Open"; the Pods table has
no right-click menu at all yet. This change adds entries to `ObjectListPanel`'s existing menu
mechanism and gives the Pods table its first one, gated by row kind, rather than building a new
menu system per panel.

## Goals / Non-Goals

**Goals:**
- Each action lives in exactly the kind(s) it applies to; no action appears on a kind it doesn't
  mean anything for.
- Port Forward's new entry points call the exact acquire path `k9s-remaining-keybindings` already
  shipped - same registry, same health/reconnect behavior, same place it's listed and stopped as
  an SSH tunnel - so the user has one mental model for "a thing forwarding a port," not two.
- Every mutating action confirms first and surfaces the cluster's own rejection message rather
  than inventing a generic one (most visible for PVC Expand, where the StorageClass decides
  feasibility, not the app).
- Create Token's result gets exactly the same handling Secret values already get: shown once,
  never persisted, never logged.

**Non-Goals:**
- Exec/attach a shell into a container. Real value, but a genuinely different UI surface (a
  terminal emulator inside a panel) and its own design problem - left for a future change rather
  than folded in here.
- Taint/untaint, suspend/resume, and pause/resume as *bulk* actions across a checked set.
  `bulk-select-list-actions` already covers the cross-kind bulk shape; adding every action here
  to that bar as well would double the surface this change needs to get right before either
  ships. A follow-up can promote any of these to bulk once the single-row version is proven.
- A generic "port forward to any kind" action. Only Pod and Service have a meaningful target
  port to forward to without the user first picking a Pod underneath some other kind (a Service
  forwards to one of its endpoint Pods; anything beyond that - Deployments, StatefulSets - would
  need the app to pick a Pod on the user's behalf, which is a different, fuzzier feature).

## Decisions

- **Port Forward's detail-panel and context-menu entries call the same handler the shipped
  `shift-f` keybinding does** (`PodsPanel::on_action_port_forward_pod`,
  `ObjectListPanel::on_action_port_forward_service`) rather than re-resolving the target or
  re-acquiring the forward themselves - the Pod-direct target, the Service-to-ready-endpoint-Pod
  resolution, and the "Port forwards" section of Manage Tunnels are already correct and unchanged.
- **Suspend/Resume, Pause/Resume Rollout, and taint add/remove are all simple field patches over
  `Api<DynamicObject>`**, following the same shape `bulk-select-list-actions`'s
  `k8s::resource::bulk_actions` module already establishes for cordon/uncordon
  (`spec.unschedulable`) and scale (`spec.replicas`). These live in a sibling module (or the same
  one, if it's still under its line-count budget by the time this change lands) rather than
  duplicating the patch-call boilerplate.
- **Trigger Now builds a `Job` manifest from the CronJob's `spec.jobTemplate`** client-side,
  the same transformation `kubectl create job --from=cronjob/<name>` does, and creates it via
  `Api<Job>::create` (or `Api<DynamicObject>` if staying kind-generic) - no server-side trigger
  subresource exists for CronJob, so this is the correct and only mechanism.
- **Expand validates the new size is not smaller than the current one client-side before
  confirming**, since Kubernetes does not support shrinking a PVC and rejecting that early saves
  a round trip; the StorageClass's own expansion support is still left to the cluster to accept
  or reject, not guessed at by the app.
- **Create Token reuses the Secret-value reveal/hide state shape** (`object-detail`'s existing
  "shown until hidden, panel closed, or explicitly hidden" rule) rather than inventing new
  reveal semantics for a second kind of sensitive value.
- **Open Host in Browser is pure UI** - read the Ingress's first rule's host from already-loaded
  data and hand it to the OS's "open URL" call. No cluster call, no confirmation, matching Copy
  Name(s)/Copy YAML's read-only treatment in `bulk-select-list-actions`.
- **Each action is gated by row kind in the same context-menu-building code that already exists**
  per panel, not a new generic "kind capability" registry. Nine kind-gated entries is small
  enough that an explicit per-kind match reads clearly; a capability-lookup abstraction would be
  solving a problem this change doesn't have yet.

## Risks / Trade-offs

- Trigger Now, Suspend/Resume, and Pause/Resume Rollout change workload behavior but are not
  destructive in the way Delete is; they still confirm, consistent with
  `bulk-select-list-actions`'s rule, even though the blast radius is smaller and reversible.
- PVC Expand cannot be undone (you can't shrink a PVC back down), which is a real cluster
  limitation, not an app choice - the confirmation text should say so rather than implying it's
  reversible.
- Create Token mints a credential; if the panel's reveal/hide handling has the same gap a Secret
  reveal would, the exposure is worse (a live-usable token, not a Secret-at-rest value). Treat
  this with the same care the risk callout in `bulk-select-list-actions`'s design gives its own
  first-mutating-path status.
