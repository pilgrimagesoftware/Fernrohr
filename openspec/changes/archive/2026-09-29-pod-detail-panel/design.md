# Design

## A Pod detail panel is identified by which pod, not just which kind

Every existing dockable panel is keyed by `PanelKey { target: NavTarget, context_name, namespaces
}` (see `app/src/util/shell.rs`), which is enough to deduplicate "the Pods list for cluster X" -
there is only one of those regardless of how many times it's (re)selected. A Pod detail panel
needs one more axis: *which* pod. Two different pods opened from the same Pods list are two
distinct panels; the same pod opened twice is the existing "focus, don't duplicate" case.

This means `NavTarget` gains a new variant carrying pod identity - namespace and name are enough
(a `PodSelection`-shaped value, already used for the pod-to-logs handoff in
`crate::k8s::resource::pods::SelectedPod`) - and `PanelKey`'s existing equality-based dedup
(`open_panels` lookup in `shell.rs`) covers the new variant with no new dedup logic, the same way
it already handles two different CRD kinds side by side.

## The structured field list, not a text blob

The reference (FreeLens) layout is a vertical list of labeled rows, most of them plain text, a
few with distinct rendering:

| Field | Rendering |
| --- | --- |
| Created | formatted relative + absolute timestamp, from `metadata.creation_timestamp` |
| Name | plain text |
| Namespace | a link-styled label (opens/focuses that namespace's scope - reuses the existing namespace-picker plumbing rather than inventing new navigation) |
| Labels, Annotations | wrapped chips, one per key=value pair |
| Controlled By | owner kind + name (from `metadata.owner_references`), link-styled |
| Managed Fields | collapsed by default, one row per manager with a "Show" toggle |
| Status | `status.phase` |
| Node | link-styled (from `spec.node_name`) |
| Host IPs, Pod IPs | plain text, from `status.host_ips`/`status.pod_ips` |
| Service Account | link-styled (from `spec.service_account_name`) |
| QoS Class | `status.qos_class` |
| Termination Grace Period | `spec.termination_grace_period_seconds`, formatted as a duration |
| Tolerations | collapsed by default, from `spec.tolerations` |
| Conditions | colored badges, one per `status.conditions[].type`, colored by its `status` (green for `True`, otherwise a muted/warning color) |

Every field here comes from the single `Pod` object already fetched - none of it needs a second
API call or a live watch. This is a `render`-time projection (a plain function from `&Pod` to a
list of row descriptors), the same shape as `pod_row` already does for the table, so it's testable
the same way: fixed `Pod` fixture in, asserted rows out, no GPUI harness required for the
projection logic itself (only the render call needs one).

The "link-styled" fields (Namespace, Controlled By, Node, Service Account) are cosmetic-only in
this change - they render as visually distinct (colored, underlined-on-hover) text, but are not
required to be clickable/navigable yet. Making them actually navigate to the linked
namespace/node/owner is worth doing but is a second, smaller pass once the field list itself
renders correctly; call this out explicitly as deferred rather than silently shipping dead links
styled as live ones - the tasks below implement each field as static-but-correctly-styled text,
and a follow-up task wires the click behavior.

A raw-YAML view stays available (the existing `PodDetail::Yaml` rendering, monospace, scrollable -
already correct content-wise per this change's earlier draft), reached via a toolbar toggle
alongside the structured view rather than replacing it.

## What leaves `PodsPanel`

`selected: Option<Pod>`, `detail: Option<PodDetail>`, and the two action handlers that produce
`PodDetail` values are deleted from `PodsPanel` entirely - table row selection still updates
`SelectedPod` (global, unchanged - `ShowPodLogs` still reads it), but no longer feeds an inline
detail render. `WarpNamespace` is unaffected; it doesn't touch `detail`.

## Opening: an action, not just a row click

`DescribePod`/`ShowPodYaml` already exist as `PodsPanel`-scoped actions (keybindings `d`/`y`,
shown in the shortcuts row). They're retargeted to call `open_target` (the same entry point
`resource.rs`'s double-click/context-menu "Open" already uses for resource kinds) with the new
`NavTarget` variant, instead of setting local `detail` state. A row's context menu gets an
explicit "Open" item for symmetry with resource-kind rows (section 9.2's pattern), rather than
requiring users to know the `d`/`y` keybindings exist.

## Title fix: list vs. detail is a property of the target, not an afterthought

`panel_title::title()` currently always calls `scope.target.label()`, and `DiscoveredKind::label()`
always returns the singular Kubernetes Kind (`"Pod"`) - correct for a detail view, wrong for a
list. Rather than add a boolean flag that every call site has to remember to set correctly, the
fix lives on `NavTarget` itself: `NavTarget::Kind(DiscoveredKind)` means "the list of this kind"
and titles from a new `DiscoveredKind::plural_label()` (title-cased `plural`, e.g. `"Pods"`); the
new pod-identity variant means "one specific item" and titles from `label()` (singular) plus the
item's name, e.g. `"Pod: api-v1-57d97859db-ftg5t"`. `panel_title::title()` matches on the
`NavTarget` variant rather than taking an extra parameter - the distinction is inherent to what a
panel *is*, not a detail its caller has to separately track and could get wrong.

This is a real, if small, spec correction: `resource-browser`'s existing scenarios describe list
behavior only and never stated what the tab should say, but the intent (visible in
`cluster-picker-and-navigation`'s section 10 notes on title bars) was clearly plural-for-list. The
delta spec makes that explicit rather than leaving it implicit and buggy.
