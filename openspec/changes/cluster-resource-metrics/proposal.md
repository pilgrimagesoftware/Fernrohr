# Proposal

## Why

Fernrohr shows live object state but no resource usage: no way to see a cluster's overall CPU and
memory trend, which nodes are under pressure, or whether a pod is approaching its limits without
leaving the app for a separate dashboard. `docs/architecture.md` already commits to a Prometheus
access path (apiserver proxy by default, a provider abstraction per Prometheus flavor) but nothing
built on it yet exists. This change is that plumbing plus its three consumers: a cluster-wide
overview panel, resource columns on the Node list, and usage graphs in the pod detail panel.

## What Changes

- **Metrics backbone**: a `MetricsProvider` abstraction that finds a cluster's Prometheus `Service`
  by label selector (lens/helm/helm-14/operator/stacklight/openshift presets) and runs PromQL
  range queries through the kube apiserver's service-proxy path, piggybacking the existing
  (possibly tunneled) kube client. A cluster with no detected Prometheus reports "no metrics
  source" rather than erroring the features built on it.
- **Cluster overview panel**: a new dockable panel, one per connected cluster context, graphing
  cluster-wide CPU and memory usage over a selectable time window (15m/1h/6h/24h), plus
  node-count and pod-count summary tiles.
- **Node list resource columns**: the Nodes resource-browser table gains CPU, memory, disk, and
  pod-count columns, each showing current usage against capacity (e.g. "4.2/8 cores"), sortable
  like any other column.
- **Pod detail metric graphs**: the pod detail panel gains a Metrics tab graphing the pod's CPU,
  memory, disk, and network usage over the same selectable windows, each graph marking the pod's
  aggregate request and limit (when set) as reference lines, plus one graph set per container
  so a multi-container pod's containers can be told apart.
- Every graph and the overview panel report "no metrics source" (not an error) when the cluster's
  Prometheus can't be found, and report a query failure distinctly from "no data in this window".

## Capabilities

### New Capabilities

- `cluster-metrics`: the Prometheus provider abstraction, query execution, and the cluster
  overview panel that is its primary consumer.

### Modified Capabilities

- `resource-browser`: the Nodes table gains metrics-sourced resource columns.
- `pod-detail`: the pod detail panel gains a Metrics tab.

## Impact

- `App/src/k8s/metrics/` (new): provider detection, PromQL templates per provider, the
  apiserver-proxy query client, and the `directUrl` opt-in path for OpenShift/kube-rbac-proxy
  (its own `ManagedForward`, per architecture.md).
- `App/src/ui/cluster_overview/` (new): the overview panel and its graphs.
- `App/src/k8s/resource/object_detail/sections/cluster.rs` or a sibling: Node table column
  definitions gain CPU/memory/disk/pod-count.
- `App/src/k8s/resource/pod_detail/` (existing pod detail module): new Metrics tab, reusing the
  existing tab-organization pattern (`pod-detail`'s "Structured fields are organized into tabs").
- A charting primitive (new, or a vetted existing crate) for line/area graphs with reference lines
  - evaluated in design.md.
- `dirs::cache_dir()/<id>/`: short-lived metric sample buffers, per architecture.md's existing
  persistence plan (cache, not state - safe to drop).
