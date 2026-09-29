Change in progress: cluster-picker-and-navigation
---
Session ending early (token budget). Everything below is committed and pushed - nothing
uncommitted, nothing local-only.

## State as of 2026-09-28

App branch `13-per-cluster-layout-restore` (pushed, origin in sync at `a9ad9aa`):
- Reconciled `feature/pod-panel-shortcuts` onto the (now-completed) `code-reorg` branch: fixed
  the reorg's missing `mod.rs` files, ~40 stale `crate::old_path::` references, one real
  regression (Pods panel's `.on_action` keybinding wiring got silently dropped by the reorg's
  auto-merge), and a rust-i18n/rust-i18n-macro version-lockfile mismatch.
- Landed regression tests for the reported "second window says connected but doesn't switch to
  workspace" bug - **did not reproduce**, confirmed manually too. No fix needed; closed as
  resolved-no-change (see `openspec/changes/second-window-connect-fix`).
- Fixed from your feedback: Pods table no longer forces monospace font, focus-highlight border no
  longer clipped by macOS's rounded window corner, list panels titled "Pods" not "Pod" (bug was in
  shared code, so this fixes every resource kind's list panel, not just Pods).
- `cargo fmt`/`clippy`/`test`: 185/185 passing (one pre-existing unrelated flake excluded, logged
  in `~/code/papercuts.md`: `placeholder_remembers_the_kind_it_was_opened_for`, a cross-test
  tokio-task leak).

Meta-repo: 5 OpenSpec change proposals written and validated, none yet implemented except the
quick fixes above:
- `app-menu-and-fonts` - App/Context/Edit/View/Navigate/Window/Help menu bar sourced from
  `CommandRegistry`, plus Manrope/Monaco fonts. Not started.
- `pod-detail-panel` - **the big one still open**. Current behavior: describing/YAML-ing a pod
  renders inline in the same `PodsPanel`, which is why it "looks exactly like the list panel."
  Needs a real FreeLens-style detail panel (screenshot referenced in the proposal): structured
  field list (Created, Namespace, Labels/Annotations as chips, Controlled By, Conditions as
  badges, etc.), toolbar, "Pod: <name>" title. Metrics charts explicitly deferred (needs
  Prometheus wiring that doesn't exist yet). The list-vs-detail title split (`NavTarget::list_label`
  / `DiscoveredKind::plural_label`) is already implemented as a prerequisite piece - see
  `app/src/ui/nav.rs` and `app/src/k8s/cluster/discovery.rs`.
- `pods-table-polish` - font part done; resizable/reorderable/sortable columns with a sort
  indicator still open. gpui-component's `TableState`/`Column` already support all three - this is
  flipping builder flags and implementing one `perform_sort` delegate hook, not new
  infrastructure. Watch out for `render_td`'s hardcoded positional `col_ix` match arms once
  columns become reorderable (design.md flags this).
- `logs-panel-header-and-performance` - Logs panel needs to show pod+container name in its title
  (it already tracks `current: Option<(String,String,String)>`, just doesn't surface it), and the
  hitchy scrolling needs `gpui::uniform_list` in place of the current unconditional
  one-`div`-per-line render.
- `panel-focus-highlight-inset` - done, needs your visual confirmation (task 1.2/2.2 still
  unchecked, marked "needs user confirmation").

Also still open, no code written: `resource-panel-grouping` (search/filter/organize the Resource
panel's kind list - this was NOT touched this session, don't assume it exists when testing).

## Suggested next step

Pick one of `pod-detail-panel`, `pods-table-polish`, or `logs-panel-header-and-performance` and
run `/opsx:apply` against it, or just start implementing from its `tasks.md` directly - all three
have full proposal/design/specs/tasks written and validated (`openspec validate <name> --strict`).
`pod-detail-panel` is the most user-visible gap and the one this feedback round centered on.

## Environment note

`$SSH_AUTH_SOCK` in a fresh shell points at the wrong (generic launchd) socket, not 1Password's
real agent socket - every signed git operation needs:
```
export SSH_AUTH_SOCK=~/"Library/Group Containers/2BUA8C4S2C.com.1password/t/agent.sock"
```
per-command (shell state doesn't persist across tool calls in this harness). Logged in
`~/code/papercuts.md` with more detail.
