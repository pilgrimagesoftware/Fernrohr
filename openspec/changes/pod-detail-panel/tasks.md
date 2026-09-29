# Tasks

## 1. Pod identity in navigation

- [ ] 1.1 Add a `NavTarget` variant carrying a pod's namespace+name (reusing the shape of
  `PodSelection`). Verify: a test constructs two variants for different pods and asserts they are
  unequal, and the same pod twice is equal - `PanelKey`'s existing dedup needs nothing further.

## 2. Panel title fix (list vs. detail)

- [ ] 2.1 Add `DiscoveredKind::plural_label()` (title-cased `plural`, e.g. `pods` -> `Pods`).
  Verify: a test asserts `plural_label()` for the core `Pod` kind returns `"Pods"`, and for a
  namespaced CRD plural like `widgets` returns `"Widgets"`.
- [ ] 2.2 `panel_title::title()`/`tab_name()` match on the `NavTarget` variant: `Kind(_)` (a list)
  titles from `plural_label()`; the new pod-identity variant (a single item) titles from
  `label()` plus `": <name>"`. Verify: a test asserts a Pods list panel's title is `"Pods"` and a
  Pod detail panel's title is `"Pod: <name>"`.

## 3. Structured field projection

- [ ] 3.1 Add a pure function projecting a `&Pod` into an ordered list of field rows (Created,
  Name, Namespace, Labels, Annotations, Controlled By, Managed Fields, Status, Node, Host IPs, Pod
  IPs, Service Account, QoS Class, Termination Grace Period, Tolerations, Conditions), each row
  carrying enough structure to know how to render it (plain text vs. chip list vs. badge list vs.
  collapsible). Verify: a test with a fixture `Pod` carrying multiple labels, annotations,
  tolerations, and conditions asserts every field is present and each condition's badge color
  matches its `status`.
- [ ] 3.2 Handle absent/optional fields gracefully (no owner references, no tolerations, a single
  IP) - these rows are omitted rather than rendered empty. Verify: a test with a minimal `Pod`
  (no owner, no tolerations) asserts those rows are absent, not blank.

## 4. `PodDetailPanel`

- [ ] 4.1 Add `PodDetailPanel`, constructed from a `PodSelection` and a `ClusterRegistry`
  connection lookup (same pattern as `PodsPanel::new`), fetching the single Pod via `Api::get`
  rather than a watch. Verify: a test with a fixture client asserts the panel renders the
  fetched pod's structured field list.
- [ ] 4.2 Render the structured field list from 3.1's projection as the panel's default view:
  chips for Labels/Annotations, colored badges for Conditions, a collapsed-by-default
  disclosure for Managed Fields and Tolerations. Namespace/Controlled-By/Node/Service-Account
  render in a visually distinct (link-like) style but are not yet clickable - that's follow-up
  work, not this task. Verify: a render-level test (or the closest available harness) asserts
  the chip/badge/disclosure structure is present.
- [ ] 4.3 Add a toolbar toggle switching between the structured view and the existing
  `PodDetail::Yaml` raw-manifest rendering (moved from `PodsPanel`, content unchanged - monospace,
  scrollable). Verify: a test toggles the view and asserts the panel shows YAML text instead of
  the field list.
- [ ] 4.4 Handle the pod no longer existing (a later `Api::get` returns 404): show a "no longer
  exists" state rather than erroring. Verify: a test simulates a 404 and asserts the panel shows
  that state without panicking or closing.
- [ ] 4.5 Title bar reads `"Pod: <name>"` per section 2's fix (not a separate implementation -
  this task is just confirming `PodDetailPanel` wires its `PanelScope` through the same
  `panel_title::title`/`tab_name` calls every other panel uses).

## 5. Wiring and cleanup

- [ ] 5.1 Retarget `DescribePod`/`ShowPodYaml` (currently setting `PodsPanel::detail` locally) to
  call the existing `open_target`-style entry point with the new `NavTarget` variant instead.
  Verify: a test invokes `DescribePod` on a pod row and asserts a `PodDetailPanel` opens for that
  pod, using the same connection (no reconnect).
- [ ] 5.2 Add an explicit "Open" context-menu item on Pods table rows (mirroring section 9.2's
  resource-kind row pattern), invoking the same path as `DescribePod`. Verify: a test invokes the
  context-menu action and asserts the same panel opens as the keybinding path.
- [ ] 5.3 Delete `selected: Option<Pod>` (repurposed only if still needed for `ShowPodLogs`'s
  existing `SelectedPod` global - confirm it is, since that path is unaffected by this change) and
  `detail: Option<PodDetail>` plus their handlers from `PodsPanel`; `PodsPanel::render` drops the
  inline detail block entirely. Verify: `PodsPanel` no longer references `PodDetail`; existing
  `PodsPanel` tests pass unchanged aside from the removed inline-detail assertions.
- [ ] 5.4 Reopening the same pod's detail while its panel is already open focuses it instead of
  duplicating (per spec). Verify: a test opens the same pod's detail twice and asserts only one
  panel exists.

## 6. Full verification

- [ ] 6.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` all
  pass.
- [ ] 6.2 Manual smoke test: open a Pods list (confirm its tab reads "Pods"), describe a pod
  (opens a panel titled "Pod: <name>" with the structured field list), toggle to YAML and back,
  close the Pods list panel, confirm the detail panel is unaffected.

## 7. Follow-up (not this change)

- Making Namespace/Controlled-By/Node/Service-Account fields actually navigate.
- Live CPU/Memory/Network/Filesystem metrics charts - depends on the Prometheus provider
  abstraction, which does not exist yet.
