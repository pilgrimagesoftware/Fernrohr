# Proposal

## Why

`bulk-select-list-actions` covers the actions that make sense across *any* checked set of one
kind or a compatible group of kinds - delete, restart/rollback rollout, scale, cordon/drain,
label/annotate. It deliberately stays generic. Several other useful actions only make sense for
one specific kind and don't fit that generic shape: forwarding a local port to a Pod or Service,
triggering a CronJob early, suspending a Job, pausing a Deployment's rollout, expanding a PVC,
tainting a Node, minting a ServiceAccount token, opening an Ingress host in a browser. None of
these exist today - list panels and detail panels are read-only (plus the Delete/Logs-family
actions `bulk-select-list-actions` adds), and `K8sPortForwardConfig`
(`App/app/src/forward/k8s/port_forward.rs`) already exists as infrastructure but has no caller
anywhere in the app yet.

## What Changes

Add the following kind-specific actions, each reachable from that kind's list-row context menu,
its detail panel (where one exists), and the command palette while that row or panel has focus:

- **Pod, Service - Port Forward...**: prompts for a local port (or offers the first container
  port/service port as a default) and starts a `ManagedForward` using the existing
  `K8sPortForwardConfig`/`PodPortForwardTransport` path, showing it in whatever UI already lists
  active forwards (tunnels editor's forward list, extended to include these).
- **CronJob - Trigger Now**: creates a Job from the CronJob's spec immediately, the same as
  `kubectl create job --from=cronjob/<name>`.
- **Job, CronJob - Suspend / Resume**: toggles `spec.suspend`.
- **Deployment - Pause / Resume Rollout**: toggles `spec.paused`. Not offered for DaemonSet or
  StatefulSet, which have no paused-rollout concept.
- **PersistentVolumeClaim - Expand...**: prompts for a new storage size and patches
  `spec.resources.requests.storage` upward. The cluster, not the app, decides whether the
  StorageClass allows it; a rejected expansion surfaces the cluster's own error.
- **Node - Add Taint... / Remove Taint**: prompts for key, value and effect to add a taint;
  offers each existing taint individually to remove.
- **ServiceAccount - Create Token**: requests a `TokenRequest` and shows the resulting token
  once, with the same reveal/never-persisted handling Secret values already get.
- **Ingress - Open Host in Browser**: opens the first rule's host (over https) in the system
  browser. No cluster call.

Every action that mutates the cluster (Port Forward excluded - it reads and connects, it does
not change cluster state) shows a confirmation before running, consistent with
`bulk-select-list-actions`'s confirm-before-mutate rule, and reports failure inline rather than
silently.

## Capabilities

### New Capabilities

- `resource-specific-actions`: kind-specific commands (port forward, trigger/suspend/resume,
  pause/resume rollout, expand PVC, taint/untaint, create token, open Ingress host), each scoped
  to the one or two kinds it applies to, reachable from that row's context menu, its detail panel,
  and the command palette.

### Modified Capabilities

(none - this adds new commands alongside existing read-only list and detail panels; no existing
requirement changes.)

## Impact

- `App/app/src/k8s/resource/pods/`, `object_list/`: new per-kind context-menu entries and
  commands, gated on the row's kind.
- `App/app/src/k8s/resource/pod_detail/`, `object_detail/`: new per-kind action entries in the
  kinds that have one (Pod's Port Forward; the generic object detail panel's sections for
  CronJob, Job, Deployment, PersistentVolumeClaim, Node, ServiceAccount, Ingress, Service).
- `App/app/src/forward/`: first real caller of `K8sPortForwardConfig`/`PodPortForwardTransport`;
  likely a small extension to whatever already surfaces `ManagedForward`s (the tunnels editor's
  forward list) so a Pod/Service port forward shows up the same way an SSH tunnel does.
- New shared module for the mutating calls (suspend/resume patch, pause/resume patch, PVC expand
  patch, taint patch, CronJob-to-Job creation, TokenRequest), parallel to and reusing patterns
  from `bulk-select-list-actions`'s `k8s::resource::bulk_actions` module where the underlying
  `Api<DynamicObject>` patterns overlap.
- `keymap.toml`-overridable key bindings and palette entries for every new action, each scoped to
  the panel/row's `KeyContext`.
