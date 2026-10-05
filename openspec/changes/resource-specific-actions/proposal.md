# Proposal

## Why

`bulk-select-list-actions` covers the actions that make sense across *any* checked set of one
kind or a compatible group of kinds - delete, restart/rollback rollout, scale, cordon/drain,
label/annotate. It deliberately stays generic. Several other useful actions only make sense for
one specific kind and don't fit that generic shape: triggering a CronJob early, suspending a Job,
pausing a Deployment's rollout, expanding a PVC, tainting a Node, minting a ServiceAccount token,
opening an Ingress host in a browser. None of these exist today - list panels and detail panels
are read-only (plus the Delete/Logs-family actions `bulk-select-list-actions` adds, and the
row-level actions `k9s-remaining-keybindings` already shipped).

Port Forward for Pods and Services is *not* one of these gaps anymore: `k9s-remaining-keybindings`
already wired `K8sPortForwardConfig`/`PodPortForwardTransport`
(`App/app/src/forward/k8s/port_forward.rs`) into the app as a Pods-row and Services-row command
(`shift-f`), including the Service-to-ready-endpoint-Pod resolution and a "Port forwards" section
in Manage Tunnels that lists and stops them. The one piece that shipped work left out is an entry
point for a mouse user who hasn't learned the keybinding: neither the Pod detail panel nor the
Service's object-detail section offers Port Forward, and neither row's context menu does either -
this change adds those entry points onto the existing acquire path, not a new one.

## What Changes

Add the following kind-specific actions, each reachable from that kind's list-row context menu,
its detail panel (where one exists), and the command palette while that row or panel has focus:

- **Pod, Service - Port Forward**: adds the action to the Pod detail panel, the Service's
  object-detail section, and both rows' context menus, each a thin call into the Port Forward
  path `k9s-remaining-keybindings` already shipped (same port-choice prompt when a target exposes
  more than one port, same `ManagedForward` acquisition, same "Port forwards" section of Manage
  Tunnels). No new transport, registry wiring, or Service-resolution logic - that already exists
  and is reused as is.
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
  kinds that have one (Pod's Port Forward, calling the existing `k9s-remaining-keybindings` acquire
  path rather than a new one; the generic object detail panel's sections for CronJob, Job,
  Deployment, PersistentVolumeClaim, Node, ServiceAccount, Ingress, Service).
- New shared module for the mutating calls (suspend/resume patch, pause/resume patch, PVC expand
  patch, taint patch, CronJob-to-Job creation, TokenRequest), parallel to and reusing patterns
  from `bulk-select-list-actions`'s `k8s::resource::bulk_actions` module where the underlying
  `Api<DynamicObject>` patterns overlap.
- `keymap.toml`-overridable key bindings and palette entries for every new action, each scoped to
  the panel/row's `KeyContext`.
