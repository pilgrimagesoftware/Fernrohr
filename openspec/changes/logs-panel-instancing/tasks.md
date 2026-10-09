# Tasks

## 1. Per-pod logs target

- [x] 1.1 Add `NavTarget::PodLogs(PodRef)`, keyed by the pod as `NavTarget::Pod` is. Verify: two pods'
  logs open two panels and the same pod's twice opens one.
- [x] 1.2 A `LogsPanel` for a `PodLogs` scope is pinned to its pod with no `SelectedPod` observer;
  the shared panel keeps today's reactive construction. Another container of the pod switches its
  panel. Verify: a pinned panel ignores a later selection; a second container reuses the panel.
- [x] 1.3 A pinned panel saves its pod and container and restores pinned, keyed as `PodLogs`.
  Verify: the dump's pod fields, `pinned_from_state`, and `panel_key` for a restored pod panel.

## 2. Preference and dispatch

- [x] 2.1 Add `UiConfig.logs_panels` (`"per_pod"` by default, or `"reuse"`) as a live, saved
  preference with two palette commands. Verify: TOML round-trip, storage by name, the default.
- [x] 2.2 `ShowLogs` opens per the preference; a new `ShowLogsFlipped` the other way, once - bound
  to `shift-l` in the Pods list and a pod's detail (outside text fields), a Pods row menu item, and
  the palette, with hints naming what the flip does. Verify: per pod by default, the flip opens the
  shared panel, the reuse preference turns both around, and keystroke tests for `shift-l` in both
  panels.
- [x] 2.3 A Settings section (Panels) sets the preference from the keyboard. Verify: the section by
  command and sidebar button, and its buttons by Tab and Space.

## 3. Full verification

- [x] 3.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` all
  pass.
- [x] 3.2 Manual smoke test: open two pods' logs with `l` and confirm each has its own panel; `l` on
  the first again focuses it; `shift-l` on a third opens the shared panel; set Settings → Panels to
  one shared panel and confirm `l` now retargets the shared panel and `shift-l` opens a pod's own.

## Notes

- Implemented in pilgrimagesoftware/Fernrohr-App#147.
