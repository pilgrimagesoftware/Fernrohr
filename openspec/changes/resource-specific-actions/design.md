# Design

## Context

`bulk-select-list-actions` (in-flight alongside this change) adds a checked-set + bulk-action-bar
layer for actions that apply uniformly across one kind or a compatible group of kinds: delete,
restart/rollback rollout, scale, cordon/drain, label/annotate, view logs. This change is its
single-row counterpart for actions that only ever make sense for one specific kind and would be
nonsensical to generalize - you don't "scale" a ServiceAccount or "taint" a Service. These belong
on that one row's context menu and that kind's detail panel, not a bulk bar.

Three things already exist that this change is the first to actually call:
- `K8sPortForwardConfig` / `PodPortForwardTransport` (`App/app/src/forward/k8s/port_forward.rs`)
  - built, but with no caller anywhere in the app.
- `ManagedForward`'s reference-counted lifecycle/health-check/teardown (`managed-forward`
  capability) - already used for SSH tunnels and the cluster's own apiserver access; Port
  Forward is a second *kind* of forward target riding the same lifecycle, not new lifecycle code.
- `kube-client`'s `create_subresource("token", ...)` helper - present in the dependency, unused
  by the app.

Every list row and detail panel today is read-only except for the actions
`bulk-select-list-actions` adds and the single-object Copy/reveal actions `object-detail`/
`pod-detail` already have. Context menus already exist per panel (Pods table's: Quick Look, Open
Details, Logs, YAML; `ObjectListPanel`'s row menu) - this change adds entries to them, gated by
row kind, rather than building a new menu system.

## Goals / Non-Goals

**Goals:**
- Each action lives in exactly the kind(s) it applies to; no action appears on a kind it doesn't
  mean anything for.
- Port Forward reuses `ManagedForward` end to end - same registry, same health/reconnect
  behavior, same place it's listed and stopped as an SSH tunnel - so the user has one mental
  model for "a thing forwarding a port," not two.
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

- **Port Forward targets a Pod directly; a Service's Port Forward picks one of its ready
  endpoint Pods and forwards to that.** `K8sPortForwardConfig`/`PodPortForwardTransport` are
  already Pod-shaped (confirmed in `forward/k8s/port_forward.rs`), so Service's action is a thin
  wrapper: resolve the Service's `Endpoints`/`EndpointSlice` to one ready Pod, then reuse the
  exact same `ManagedForward` path Pod's own action uses. No second transport is written.
- **Port Forward shows up in whatever UI already lists `ManagedForward`s** (the tunnels editor's
  forward list, per the `managed-forward` capability), rather than building a second
  "active forwards" surface. A Pod-forward row there needs only a label distinguishing it from
  an SSH tunnel and the same stop control tunnels already have.
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

- Port Forward is this change's most structurally significant piece: it's the first UI caller of
  existing forward infrastructure, and getting the Service-to-Pod resolution wrong (picking a
  non-ready or terminating endpoint) would make the feature flaky in exactly the cases it's
  meant to help with. Worth its own focused task and test rather than being treated as "just
  another patch."
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
