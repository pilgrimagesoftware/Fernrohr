# Tasks

## 1. Pod identity in navigation

- [ ] 1.1 Add a `NavTarget` variant carrying a pod's namespace+name (reusing the shape of
  `PodSelection`). Verify: a test constructs two variants for different pods and asserts they are
  unequal, and the same pod twice is equal - `PanelKey`'s existing dedup needs nothing further.

## 2. `PodDetailPanel`

- [ ] 2.1 Add `PodDetailPanel`, constructed from a `PodSelection` and a `ClusterRegistry`
  connection lookup (same pattern as `PodsPanel::new`), fetching the single Pod via `Api::get`
  rather than a watch. Verify: a test with a fixture client asserts the panel renders the
  fetched pod's description.
- [ ] 2.2 Move `PodDetail` (`Description`/`Yaml`) and its rendering block verbatim from
  `PodsPanel` into `PodDetailPanel`, with a mode toggle (description vs YAML) the panel owns.
  Verify: existing description/YAML rendering tests migrate and still pass against the new panel.
- [ ] 2.3 Handle the pod no longer existing (a later `Api::get` returns 404): show a "no longer
  exists" state rather than erroring. Verify: a test simulates a 404 and asserts the panel shows
  that state without panicking or closing.
- [ ] 2.4 Give `PodDetailPanel` a title bar following the section-10 convention: kind "Pod", plus
  the pod's namespace/name. Verify: a test asserts the title bar reflects the scoped pod.

## 3. Wiring and cleanup

- [ ] 3.1 Retarget `DescribePod`/`ShowPodYaml` (currently setting `PodsPanel::detail` locally) to
  call the existing `open_target`-style entry point with the new `NavTarget` variant instead.
  Verify: a test invokes `DescribePod` on a pod row and asserts a `PodDetailPanel` opens for that
  pod, using the same connection (no reconnect).
- [ ] 3.2 Add an explicit "Open" context-menu item on Pods table rows (mirroring section 9.2's
  resource-kind row pattern), invoking the same path as `DescribePod`. Verify: a test invokes the
  context-menu action and asserts the same panel opens as the keybinding path.
- [ ] 3.3 Delete `selected: Option<Pod>` (repurposed only if still needed for `ShowPodLogs`'s
  existing `SelectedPod` global - confirm it is, since that path is unaffected by this change) and
  `detail: Option<PodDetail>` plus their handlers from `PodsPanel`; `PodsPanel::render` drops the
  inline detail block entirely. Verify: `PodsPanel` no longer references `PodDetail`; existing
  `PodsPanel` tests pass unchanged aside from the removed inline-detail assertions.
- [ ] 3.4 Reopening the same pod's detail while its panel is already open focuses it instead of
  duplicating (per spec). Verify: a test opens the same pod's detail twice and asserts only one
  panel exists.

## 4. Full verification

- [ ] 4.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` all
  pass.
- [ ] 4.2 Manual smoke test: open a Pods list, describe a pod (opens a panel), view its YAML
  (same panel or a second, per design), close the Pods list panel, confirm the detail panel is
  unaffected.
