# Proposal

## Why

Selecting a pod in the Pods table currently renders its description or YAML inline, above the
table, inside the same `PodsPanel` - so the "detail view" is indistinguishable from the list
panel it was opened from (confirmed against a running build: it visibly is the same panel with a
small text block glued on top). It can't be viewed side by side with the list, moved, closed, or
maximized independently - every other panel in this app gets all of that for free from the dock.
FreeLens's pod detail view (reference screenshot on file with this change) is the bar: a
structured field list - Created, Name, Namespace, Labels, Annotations, Controlled By, Node,
Status, Conditions, and more - each rendered appropriately (chips for labels/annotations, linked
namespace/node/service-account, colored badges for conditions), not a wall of raw YAML text.

Separately, every resource panel's tab - list or detail alike - is titled from the Kubernetes
*Kind* name (`"Pod"`), because that's the only label `DiscoveredKind` computes. That's backwards
for a list ("Pods" list panel is titled "Pod") and merely incomplete for a detail view ("Pod"
panel with no indication of *which* pod). Fixing "which title for which kind of panel" is
inseparable from adding the detail panel, since a title needs to know it belongs to a list versus
a single item, and today nothing does.

## What Changes

- Opening a Pod's detail (via `DescribePod`/`ShowPodYaml`, or an explicit "Open" action on a table
  row) opens a distinct dockable "Pod" panel, scoped to that specific pod, instead of rendering
  inline inside the Pods list panel.
- The Pod detail panel renders a structured field list matching FreeLens's layout and content for
  the fields obtainable from the Pod object itself: Created, Name, Namespace (linked), Labels and
  Annotations (as chips), Controlled By (linked to its owner), Managed Fields (collapsible), Status,
  Node (linked), Host/Pod IPs, Service Account (linked), QoS Class, Termination Grace Period,
  Tolerations (collapsible), Conditions (colored badges). Raw YAML remains available as an
  alternate view (a toolbar toggle), since it's still useful for the exact-manifest case.
  **Non-goal, explicitly deferred**: FreeLens's live CPU/Memory/Network/Filesystem metrics charts.
  Those need the Prometheus provider abstraction described in `openspec/config.yaml`'s
  architecture notes, which does not exist yet - a separate change once that lands.
- Selecting a Pod that already has an open detail panel for the same cluster focuses that panel
  rather than opening a duplicate, mirroring the existing resource-kind panel de-duplication rule.
- Panel titles distinguish list from detail: a list panel is titled by the kind's plural
  ("Pods"), a detail panel by the kind's singular plus the specific item's identity
  ("Pod: api-v1-57d97859db-ftg5t"). This generalizes past Pods to every resource-kind panel's tab,
  since the underlying title computation is shared and was simply never given the distinction to
  make.
- The `ShowPodLogs` action continues to target the existing Logs panel/selection mechanism
  unchanged; this proposal only concerns the description/YAML/structured detail views.

## Capabilities

### New Capabilities
- `pod-detail`: a dockable panel showing one Pod's structured detail (fields, YAML), opened from
  a Pods list selection.

### Modified Capabilities
- `resource-browser`: adds the list-vs-detail panel title distinction (a resource panel showing a
  list of a kind is titled by its plural name; this did not previously exist as a stated
  requirement, and today's actual behavior - singular for both - contradicts what section 10 of
  `cluster-picker-and-navigation` intended, so it lands here as a correction).

## Impact

- `app/src/k8s/resource/pods.rs`: `PodDetail`/`on_action_describe_pod`/`on_action_show_pod_yaml`
  move from `PodsPanel` state into a new panel type; `PodsPanel::render` drops the inline detail
  block.
- `app/src/k8s/cluster/discovery.rs`: `DiscoveredKind::label()` gains a plural counterpart (or an
  explicit list/detail parameter) for the title fix.
- `app/src/ui/panel/title.rs`: `title()`/`tab_name()` take whether the scope is a list or a single
  item into account.
- Dock/panel registration (`register_panel`, `PANEL_KEY_CONTEXT`-style key context) gains a new
  panel kind, following the existing `PlaceholderPanel`/`LogsPanel` pattern for panel-open and
  panel-focus-if-already-open; the new panel needs its own `dump`/restore entry in saved dock
  layouts.
