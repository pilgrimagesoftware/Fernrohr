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

## Detail rendering itself moves, not gets rewritten

`PodDetail` (`Description(String)` / `Yaml(String)`) and the rendering already built for it in
`PodsPanel::render` (the `.mb_3()...max_h(px(240.))...font_family(mono)...overflow_scrollbar()`
block) is already correct content-wise - the fix here is *where* it lives, not what it looks like.
The new `PodDetailPanel`:

- Is constructed with a `PodSelection` and the cluster's connection (same
  `ClusterRegistry::connection` lookup every other panel uses), not a live reference back into
  `PodsPanel`.
- Fetches the one Pod object it needs directly (a single `Api::get`, not a watch - a detail view
  for one already-selected object doesn't need to track every cluster mutation live the way a list
  panel does; if the pod is deleted while the panel is open, the panel shows a "no longer exists"
  state rather than erroring).
- Owns its own `on_action_describe_pod`/`on_action_show_pod_yaml`-equivalent toggle between the
  two detail modes, and its own title bar (kind="Pod", plus the pod's namespace/name, following
  the existing title-bar convention from section 10).

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
