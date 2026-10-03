# Tasks

## 1. Metrics backbone

- [ ] 1.1 Add `k8s::metrics::provider` with a `MetricsProvider` enum/trait covering lens, helm,
  helm-14, operator, and stacklight presets, each as a label selector plus its PromQL templates
  for CPU/memory/disk/network-by-pod and CPU/memory-by-node; verify with unit tests that each
  preset's selector matches a fixture `Service` list and that no preset matches an empty list.
  (openshift's `directUrl` mode is out of scope per design.md's Non-Goals.)
- [ ] 1.2 Add `detect_provider(client) -> Option<MetricsProvider>` that tries each preset in turn
  against the cluster's `Service` list, and cache the result on `ClusterSession`, detected once
  per connection; verify with a test that detection runs once even across multiple callers.
- [ ] 1.3 Add `query_range(client, provider, promql, window) -> Result<Series, MetricsError>`
  issuing the PromQL range query through the apiserver service-proxy path, with `MetricsError`
  distinguishing `NoSource`, `QueryFailed`, and a successful empty result (no-data-yet is not an
  error); verify with tests covering a successful series, an HTTP error, and a cluster with no
  detected provider short-circuiting to `NoSource` without a request.
- [ ] 1.4 Add a short-lived sample buffer under `dirs::cache_dir()/<id>/`, per
  architecture.md's existing cache-dir convention; verify a missing or corrupt buffer file falls
  back to re-querying rather than failing.

## 2. Cluster overview panel

- [ ] 2.1 Read gpui-kit 0.7's `plot`/`chart` module to resolve design.md's open question on
  reference lines, then build the overview panel on `component::chart::{line_chart, area_chart}`
  graphing cluster-wide CPU and memory over a selectable window (15m/1h/6h/24h), plus node-count
  and pod-count tiles; verify with a snapshot/behavior test that switching windows redraws both
  graphs from freshly queried data.
- [ ] 2.2 Register an `open_cluster_overview` command (one panel per connected cluster context,
  reselecting an already-open one focuses it, matching the existing per-object detail-panel
  focus-or-open pattern), keybound and in the command palette; verify opening twice for the same
  context focuses the existing panel rather than opening a second.
- [ ] 2.3 Wire the "no metrics source" and "query failed" states from task 1.3 into the panel's
  graphs and tiles; verify both states render distinctly from a normal loading state.
- [ ] 2.4 Cover keyboard operability: every time-window option reachable and selectable without a
  mouse, with its key shown, per `.claude/rules/keyboard-first.md`; verify with a
  `simulate_keystrokes` test.

## 3. Node list resource columns

- [ ] 3.1 Add CPU, memory, disk, and pod-count column definitions to the Nodes resource-browser
  table, CPU/memory/disk sourced from `query_range` (or an instant-query variant, if the metrics
  backbone needs one for "current value" rather than a series) and pod-count from the existing
  pod Store filtered by node (reusing `node-detail-pod-list`'s field-selector approach if that
  change has landed, or the pod Store otherwise); verify with a test asserting each column's
  value and that it participates in the existing sort mechanism.
- [ ] 3.2 Show "usage unavailable" in the CPU/memory/disk columns when the cluster has no
  detected metrics source, distinct from a zero value; verify with a test against a cluster
  fixture with no matching provider.

## 4. Pod detail Metrics tab

- [ ] 4.1 Add the Metrics tab as position 7 in the pod detail panel's tab order, updating the
  `Tab keys follow tab order` behavior to include it; verify a keyboard test that pressing `7`
  shows the tab.
- [ ] 4.2 Graph the pod's aggregate CPU, memory, disk, and network usage over the selectable
  window, with request/limit reference lines on the CPU and memory graphs when the pod sets them
  (aggregated across containers) and no reference lines on disk/network; verify with tests
  covering: requests and limits both set, a request with no limit, and disk/network showing no
  reference lines.
- [ ] 4.3 For a pod with more than one container, add one labeled graph set per container with
  that container's own request/limit lines; verify a single-container pod shows no separate
  per-container section (would duplicate the aggregate) and a two-container pod shows two labeled
  sets.
- [ ] 4.4 Wire the "no metrics source" and "query failed vs. no data yet" states into the Metrics
  tab; verify with tests for each of the three states, including a just-created pod showing "no
  data yet" rather than an error for the portion of the window before it existed.

## 5. Integration and documentation

- [ ] 5.1 Run `cargo fmt && cargo clippy -- -D warnings && cargo test` and fix any failures.
- [ ] 5.2 Update `docs/architecture.md`'s Prometheus section if the implemented shape (provider
  caching on `ClusterSession`, the three-state `MetricsError`) diverges from what it currently
  describes, so the doc stays the source of truth for the decision it already commits to.
