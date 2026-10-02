# Proposal

## Why

The Resource panel lists every kind the cluster reports, but only Pods open a list. Every other
kind, including Deployments, Services and ConfigMaps, opens a "has no panel implementation yet"
placeholder. The generic object-detail panel already exists, but you can only reach it by
following a link from a panel that's already open. So there's no way to browse all Services and
open one. This is the main gap left in day-to-day use.

## What Changes

- Every discovered kind opens a live list panel instead of the placeholder. The panel has the
  same namespace scoping, filter, sort, column resize/reorder, shared watch and keyboard behavior
  as the Pods table.
- Every list shows Name, Age and, for namespaced kinds, Namespace. Built-in kinds also get the
  columns `kubectl get` shows by default (Deployment Ready/Up-to-date/Available, Service
  Type/Cluster IP/Ports, PVC Status/Volume/Capacity, and so on).
- Opening a list row (Enter or double-click) opens that object's detail panel. Pods keep their
  dedicated pod-detail panel; every other kind uses the object-detail panel.
- object-detail gains kind-specific sections for Service, Ingress, Endpoints, EndpointSlice,
  NetworkPolicy, Namespace, PersistentVolume, StorageClass, CronJob, Role, ClusterRole,
  RoleBinding and ClusterRoleBinding.
- The placeholder panel stays only as the restore fallback for a saved layout whose kind is no
  longer discovered.

## Capabilities

### New Capabilities

(none)

### Modified Capabilities

- `resource-browser`: list panels for every discovered kind, not just Pods, with per-kind columns
  for built-in kinds; rows open the object's detail panel.
- `object-detail`: reachable from a list row as well as from a link; kind-specific sections cover
  more built-in kinds.

## Impact

- App: new dynamic-object list panel next to `k8s/resource/pods/`; `ui/nav.rs` routing (which
  currently gates on `has_concrete_panel`); panel restore in `util/shell/panels.rs`; new section
  modules under `k8s/resource/object_detail/sections/`.
- One shared watch per (cluster, kind) for every kind a panel opens, so more concurrent watches
  when many lists are open. Same lifecycle as the Pods watch.
- No new crates. `kube::Api<DynamicObject>` and `k8s-openapi` already cover this.
