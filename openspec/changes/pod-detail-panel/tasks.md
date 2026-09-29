# Tasks

## 1. Pod identity in navigation

- [x] 1.1 Added `NavTarget::Pod(PodRef { namespace, name })`. `PanelKey`'s existing equality-based
  dedup covers it with no new logic - `a_pod_detail_panel_is_keyed_by_which_pod` and
  `two_pods_are_different_targets_and_one_pod_is_one_target` cover both branches.

## 2. Panel title fix (list vs. detail)

- [x] 2.1 `DiscoveredKind::plural_label()` added (title-cased `plural`).
- [x] 2.2 `panel_title::title()`/`tab_name()` dispatch on the `NavTarget` variant: `Kind(_)`
  titles from `plural_label()` ("Pods"), `NavTarget::Pod(_)` from `label()` plus `": <name>"`
  ("Pod: <name>"). Covered by `a_pods_detail_panel_names_its_pod` and the shell-level
  `every_resource_panel_carries_its_title_bar`.

## 3. Structured field projection

- [x] 3.1 `pod_fields(&Pod, Timestamp) -> Vec<PodField>` implemented in `pod_detail.rs`, covering
  every field listed plus - added after review against the full FreeLens layout rather than just
  the one guidance screenshot - Containers, Init Containers, and Volumes (section 3.3).
  `every_structured_field_is_projected_in_order` covers ordering and presence.
- [x] 3.2 Absent fields omitted, not rendered blank - covered by the minimal-pod branch of the
  same test plus `a_container_with_no_status_yet_still_gets_a_row`.
- [x] 3.3 (added, not in the original list) Containers/Init Containers: per-container image,
  ready state, restart count, ports, resource requests/limits, joined from `spec.containers`
  (or `spec.init_containers`) with `status.container_statuses` by name via
  `summarize_containers`. Volumes: name + source type via `format_volume`. Covered by
  `containers_join_spec_and_status_by_name`, `a_container_with_no_status_yet_still_gets_a_row`,
  `volumes_are_named_and_typed`.

## 4. `PodDetailPanel`

- [x] 4.1 `PodDetailPanel::new` fetches via `Api::get` (no watch), same `ClusterRegistry`
  connection-lookup pattern as `PodsPanel::new`.
- [x] 4.2 Structured field list is the default view: chips (Labels/Annotations), colored badges
  (Conditions), container cards (Containers/Init Containers), and a collapsed-by-default
  disclosure - generalized from two hardcoded bools to a label-keyed `open_sections` set once
  Volumes needed its own independent collapse state.
- [x] 4.3 Toolbar toggle between structured and YAML views. `y` (`ShowPodYaml`) opens directly
  into YAML per the explicit decision recorded when this was flagged mid-implementation - not the
  structured-default-then-toggle reading originally assumed.
- [x] 4.4 404 -> `PodDetailState::NotFound`, rendered as "This pod no longer exists," panel stays
  open.
- [x] 4.5 Title bar reads `Pod: <name>` (section 2's fix, confirmed via the shared
  `panel_title::title` path - no separate implementation needed).

## 5. Wiring and cleanup

- [x] 5.1 `DescribePod`/`ShowPodYaml` emit `ShowPodDetail`/`ShowPodDetailYaml` (two actions, not
  one with an argument - gpui's actions are unit structs, and a single action would leave `d`/`y`
  indistinguishable at the dispatch end, which is exactly the discoverability bug section 6
  below caught). `MainWindow` resolves the pod from `SelectedPod` and calls `open_target`.
- [x] 5.2 Row context-menu "Open" added, invoking the same path as `DescribePod`.
- [x] 5.3 `PodsPanel::{selected, detail}` and their inline render deleted; `WarpNamespace`/
  `ShowPodLogs` now read `SelectedPod` directly.
- [x] 5.4 Reopening the same pod's detail focuses the existing panel -
  `a_pod_detail_panel_is_keyed_by_which_pod`.

## 6. Full verification

- [x] 6.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` clean; `cargo test`
  211/211 (a `tunnel::store` keychain-lock test intermittently hangs under parallel execution on
  this machine independent of this change - pre-existing, per `10-panel-title-bar`'s 8c0a1b7 -
  skipped when it does, not counted as a failure).
- [ ] 6.2 Manual smoke test: open a Pods list (confirm its tab reads "Pods"), describe a pod
  (opens a panel titled "Pod: <name>" with the structured field list, including Containers and
  Volumes), toggle to YAML and back, confirm `y` opens straight into YAML, confirm the panel's
  focus border stays visible while a child (a table row) has focus, close the Pods list panel,
  confirm the detail panel is unaffected. **Needs a real interactive desktop session.**

## 7. Follow-up (not this change)

- Making Namespace/Controlled-By/Node/Service-Account/container-image fields actually navigate.
- Live CPU/Memory/Network/Filesystem metrics charts - depends on the Prometheus provider
  abstraction, which does not exist yet.
- Per-tab close button and namespace-picker title-bar sharing - see
  `openspec/changes/per-tab-close-button` and the namespace-picker relocation landed as a
  follow-up fix in this change's implementation commits, not tracked here originally.
- Tabbed structured view - `openspec/changes/pod-detail-tabs`.
- Per-container expansion beyond the summary card - `openspec/changes/container-detail-expansion`.

## 8. Second round of fixes (from manual review before 6.2 could run)

- [x] 8.1 Pods table row double-click (`TableEvent::DoubleClickedRow`) now opens the pod's detail
  panel - previously unhandled, so nothing opened detail except the row's context menu (and `d`,
  itself unbound until the keybinding fix below).
- [x] 8.2 `d`/`w`/`l`/`y` were printed in the Pods panel's hint bar but never bound to a real
  keystroke (`shell::init` only bound NewWindow/Palette/Pods/Logs) - fixed via
  `pods::panel_bindings()`, registered in `PANEL_KEY_CONTEXT`.
- [x] 8.3 `focus_frame` asked whether the panel's own focus handle was the focused element, which
  stops being true the moment a child (a table row) takes focus - the border went dark exactly
  when the panel was in use. Now checks `contains_focused`.
- [x] 8.4 YAML/structured toggle relocated into the panel body with its own keybind, per section 2
  of the namespace-picker relocation pattern (`PodDetailPanel`'s own key context, `y`).
