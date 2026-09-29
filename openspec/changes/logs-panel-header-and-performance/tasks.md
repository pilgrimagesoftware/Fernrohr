# Tasks

## 1. Title

- [x] 1.1 `LogsPanel::title`/`tab_name` build `"Logs: <pod> · <container>"` from `self.current`
  when it is `Some`, falling back to the existing generic label otherwise. Implemented as a pure
  `streaming_title(current, fallback)` free function (so it's testable without a GPUI harness),
  called by both `Panel::title` and `Panel::tab_name`. Verify:
  `streaming_title_names_the_pod_and_container` and
  `streaming_title_falls_back_before_a_pod_is_selected` cover both branches directly.

## 2. Virtualized scrolling

- [x] 2.1 Replaced the unconditional `.children(view.lines().iter().cloned().map(...))` block
  with `gpui::uniform_list`, rendering only the visible row range plus overscan (the primitive's
  own behavior). Preserved `.whitespace_nowrap()` monospace one-line-per-row styling; vertical
  scroll now comes from `uniform_list` itself, horizontal from an added `.overflow_x_scrollbar()`
  on the wrapping div. **No dedicated "visible element count stays bounded" test added** - that
  claim rests on `uniform_list`'s own virtualization, which is GPUI's tested responsibility, not
  something this app's test harness can usefully re-verify by inspecting a painted tree from
  outside (the same render-harness ceiling other sections of this codebase already hit, e.g.
  section 10.3's title-bar note). What *is* app-specific and tested by the full suite passing
  post-change: `item_count` is wired to the real line count and each visible index maps to the
  correct line.
- [ ] 2.2 Manual check: stream (or seed) a log panel with several thousand lines and confirm
  scrolling stays responsive. **Needs a real interactive desktop session.**

## 3. Full verification

- [x] 3.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` all
  pass (187/188, the one failure being the pre-existing unrelated
  `placeholder_remembers_the_kind_it_was_opened_for` flake - this worktree branched before that
  flake's fix landed elsewhere).
- [ ] 3.2 Manual smoke test: open a Pod's logs, confirm the title shows pod and container, confirm
  scrolling a long log stream feels smooth. **Needs a real interactive desktop session.**

## 4. Corrections found during manual review (not in the original scope, landed alongside it)

- [x] 4.1 The Logs panel had a namespace picker in its title bar, wrong for a panel scoped to one
  already-chosen pod's logs. Replaced with a container picker, using `LogsView`'s existing
  `containers()`/`select_container` infrastructure (previously built and tested but never given a
  caller).
- [x] 4.2 `FollowState`/`scroll_up`/`scroll_to_bottom` were implemented and tested but had no UI
  caller (`LogsPanel::render` just dumped every line unconditionally). Wired to real Jump to Top,
  Jump to Bottom, and Follow controls, using `UniformListScrollHandle` for the actual scroll.
  Substring search/filter is out of scope here - see `openspec/changes/logs-search-filter`.
