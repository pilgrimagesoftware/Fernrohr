# Design

## Context

`docs/architecture.md` already commits to the metrics access path: query through the kube
apiserver's service-proxy by default (reusing the existing, possibly tunneled, `kube::Client`),
a provider abstraction that finds Prometheus by label selector per flavor (lens, helm, helm-14,
operator, stacklight, openshift), and an opt-in `directUrl` mode for OpenShift/kube-rbac-proxy
that needs its own `ManagedForward`. None of that is built yet. gpui-kit 0.7
(`component::chart::{line_chart, area_chart, bar_chart, ...}`, built on its own `plot` module
with `Grid`, `Scale`, hover tooltips) already ships line and area charts with axis labels and
hover crosshairs - no new charting dependency is needed.

Pod detail (`App/src/k8s/resource/object_detail` sibling for pods) already organizes fields into
positional, number-keyed tabs (`pod-detail`'s "Structured fields are organized into tabs" /
"Tab keys follow tab order"); the Metrics tab is one more entry in that existing pattern. The
Nodes table is the same `resource-browser` table every other kind uses; adding columns is
configuring that table's column set for the Node kind, not new table infrastructure.

See proposal.md for why; see the three spec deltas for exact requirements.

## Goals / Non-Goals

**Goals:**
- One metrics backbone (provider detection + range query execution) consumed identically by the
  overview panel, the Node columns, and the pod Metrics tab - no feature gets its own query path.
- Every consumer degrades the same way with no metrics source: say so, don't error or show a
  misleading zero.

**Non-Goals:**
- Historical retention beyond what the cluster's own Prometheus retains - the app queries live,
  it does not run its own time-series store.
- Alerting or threshold notifications on usage - this change is observability, not alerting.
- A `directUrl` (OpenShift/kube-rbac-proxy) implementation in this change's first slice - the
  provider abstraction is designed to support it (per architecture.md), but wiring an opt-in
  direct-URL `ManagedForward` is deferred to a follow-up once the apiserver-proxy path is proven;
  tasks.md tracks the apiserver-proxy path only.

## Decisions

**One `MetricsProvider` trait, one per connected cluster, cached per `ClusterSession`.**
Detection (which provider matches, if any) runs once per cluster connection and is cached
alongside the existing `ClusterSession` state (`kube::Client`, watchers, Store cache per
architecture.md), not re-run per query. A cluster with no match caches that fact too, so the
three consumers don't each repeat a failed detection.

**Range queries are a thin wrapper over the existing kube client, not a new HTTP stack.**
`query_range(client, provider, promql, window) -> Result<Series, MetricsError>` issues the
service-proxy request through the same `kube::Client` already used for every other API call -
no `reqwest` or other HTTP crate, and no separate auth/TLS handling, since the proxy path rides
the cluster's existing credentials.

**`MetricsError` distinguishes "no source", "query failed", and "no data yet" as three states, not
one `Option`/`Result` collapse.** Every spec delta's "query failure is distinct from no data"
requirement needs this distinction visible to the UI layer, not just logged - collapsing them
into a single error variant would force each of the three consumers to re-derive the distinction
independently (and risk drifting on which is which).

**Reference lines on existing gpui-kit charts: confirm the primitive before committing to it.**
The exact way to draw a fixed horizontal reference line (request/limit) on a `LineChart`/
`AreaChart` needs checking against gpui-kit 0.7's actual `plot`/`chart` API (a second thin series,
a dedicated annotation primitive, or an owned draw-after-plot layer) - see Open Questions. The
requirement (a visible line at a fixed value) does not change based on the answer, only the
implementation inside the Metrics tab's rendering code.

**Node columns reuse the resource-browser table's existing column mechanism.** `resource-browser`
already supports resizable, reorderable, sortable columns generically; CPU/memory/disk/pod-count
are four more column definitions for the Node kind specifically, sourced from the cached
`MetricsProvider` (pod-count from the existing pod Store, not metrics - it needs no Prometheus).
No change to the table widget itself.

**Metric sample buffers are cache, not state.** Per architecture.md's existing persistence plan,
recent samples (if buffered at all, to smooth a flapping query) live under
`dirs::cache_dir()/<id>/`, safe to drop on restart - a graph that loses its buffer on restart
just re-queries, it does not need to survive a restart the way workspace layout does.

## Risks / Trade-offs

[A cluster's Prometheus uses non-standard metric names (a custom exporter, an unusual
`kube-state-metrics` version)] → Each provider carries its own PromQL templates (per
architecture.md); a cluster matching no provider's label selector is reported as "no metrics
source" rather than silently showing wrong numbers from a mismatched template.

[Range queries on a slow or far-tunneled cluster add latency to panels that otherwise feel
instant] → Queries run off the render path (existing async-fetch pattern, like object detail's
event fetch) and each consumer shows its own loading state; a slow metrics query never blocks the
object data that panel already shows.

[Four new resource-browser columns on every Nodes table, even for users who never look at them] →
Accepted: columns are what the table mechanism already supports, and sortable usage columns are
the whole point of this slice of the request.

## Migration Plan

Additive only: a new capability and two new fields/tabs on existing panels. No existing data
shape changes; nothing to migrate for a user who never opens the overview panel or Metrics tab.

## Open Questions

- Does gpui-kit 0.7's `chart`/`plot` module already expose a reference-line or annotation
  primitive, or does the request/limit line need a second thin series layered on top? Resolve by
  reading the vendored `plot` module during implementation (task 2.1) - it changes an
  implementation detail inside the Metrics tab, not the requirement that the line be shown.
