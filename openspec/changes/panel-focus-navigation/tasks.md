# Tasks

## 1. Focus stops and stepping

- [ ] 1.1 Add `app/src/ui/panel/focus.rs` (declared in `ui/panel/mod.rs`) with `step(len, current,
  direction)`. It wraps at both ends; with nothing focused, `Next` gives the first stop and
  `Previous` the last; zero stops gives `None`. Verify with unit tests in `ui/panel/focus/tests.rs`
  covering wrap-around, previous-undoes-next, no current stop, one stop, and none.
- [ ] 1.2 Add `focus_stops`. It returns the Resource panel (when shown), then each open dock region
  in `Left, Center, Right, Bottom` order: pre-order over the region's tree, taking each tab group's
  displayed panel and skipping panels that aren't `visible`. While zoomed, only the zoomed group's
  displayed panel counts. Verify with tests over a real `DockSkin` dock: split order, a collapsed
  dock skipped, a hidden tab not a stop, and zoom narrowing to one dock stop.

## 2. Next / previous panel commands

- [ ] 2.1 Register `panel.focus_next` ("Focus Next Panel", `cmd-]`) and `panel.focus_previous`
  ("Focus Previous Panel", `cmd-[`). Both are global, in `MenuSlot::Navigate`, and registered from
  `shell::register_commands`. Verify: a test that `registry.available(&[])` lists both, and a
  test over the full app registry that no other command has the same default binding in an
  overlapping context.
- [ ] 2.2 Handle both actions on `MainWindow`, next to `FocusResources`. Rebuild the stops, find
  the current stop by `contains_focused`, step, and focus the chosen stop. Verify with a
  `util/shell/tests.rs` test in the shape of `the_focus_resources_key_focuses_the_resource_panel`.
  With two dock panels open, real `cmd-]` keystrokes should move focus Resource → first panel →
  second panel → Resource (wrap), and `cmd-[` should step back.

## 3. An opened panel takes focus

- [ ] 3.1 In `shell::open_target_with_view`, after either the `select_panel` (already open) or the
  `add_panel_view` (new) arm, focus the panel's handle. Verify with shell tests. Opening a new pod
  detail through the Pods panel's `d` keystroke should leave the new panel focused. Asking for an
  already-open panel from the keyboard should bring it to the front and focus it.
- [ ] 3.2 Update doc comments that say opening a panel doesn't focus it, and the
  `panel-focus-highlight-inset` follow-up 4.1 note, to point here. Verify with a grep for stale
  wording.

## 4. Verification

- [ ] 4.1 `cargo fmt -- --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` all
  pass, and every file touched stays under the 500-line limit.
- [ ] 4.2 Manual check on a running build: with the Resource panel and two or more dock panels,
  cycle with `cmd-]` / `cmd-[` and confirm the tab underline follows. Collapse a dock and zoom a
  panel, and confirm hidden panels are skipped. Open a pod's detail with `d` and confirm it has
  focus. **Needs user confirmation.**
