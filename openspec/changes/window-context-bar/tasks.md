# Tasks

## 1. Window context model

- [ ] 1.1 Replace `WindowMode::Workspace`'s `context_name` with `contexts: Vec<String>` and
  `active: usize`, with `connection_count` derived from `contexts.len()`, and no behavior change.
  Verify: the existing shell, picker, and restore tests pass unchanged.
- [ ] 1.2 Add `ClusterRegistry::hold`/`release` keyed by window, removing the session on the last
  release. Verify: tests for two windows holding one context (one session), one releasing (session
  kept, watches live), and the last releasing (session gone and its tunnel's `ForwardRegistry` entry
  released, using a fake forward).
- [ ] 1.3 Take a hold when a window enters a workspace or restores, and release on window close.
  Verify: a GPUI test opens two windows on one context, closes one, and asserts the session remains;
  closing the other removes it.

## 2. Persistence

- [ ] 2.1 Add `contexts` to `WindowLayout` with a serde default, derived from saved panels'
  `cluster_context` when absent, and restore every listed context. Keep per-context dock layouts only
  for single-context windows. Verify: round-trip tests for a two-context window, a legacy file with
  no `contexts`, and a window whose second context has no panels.

## 3. Context bar

- [ ] 3.1 Add `ui/context_bar.rs` rendered under the title bar in workspace mode, with one chip per
  context showing its name, bound tunnel name, and health dot (via `ClusterRegistry::health` and
  `severity` if `connection-status-bar` has landed, otherwise no dot). A chip click sets `active`,
  kept in sync both ways with the Resource panel's cluster dropdown. Verify: GPUI tests for chip
  labels, and for a chip click and a dropdown change each updating the other.
- [ ] 3.2 Add the "+" popover, wrapping the picker filtered to unused contexts. Choosing one takes a
  hold, connects or reuses the session, opens a Pods panel, and sets `active`. A failed connect shows
  on the chip and opens no panel. Verify: GPUI tests for filtering, for adding a context already
  connected in another window (no second session), and for the failure path with a fake connection.
- [ ] 3.3 Add Disconnect to the chip menu, with a confirmation stating the panel count and the other
  windows' use; on confirm, close that context's panels, release the hold, and fall back to the
  picker when none remain. Verify: GPUI tests for confirm, cancel, the "stays connected in 1 other
  window" text, and last-context-to-picker.

## 4. Status bar alignment

- [ ] 4.1 If `connection-status-bar` has landed, point its item source at the window's `contexts`.
  Verify: its GPUI tests pass against a two-context window, and Disconnect removes the item.

## 5. Full verification

- [ ] 5.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` all
  pass.
- [ ] 5.2 Manual check against QA: in one window add `greedygoat` and `carefulcrab`, arrange panels for
  both, quit and relaunch (both restored), open a second window with `greedygoat`, disconnect it from
  the first window (confirmation mentions the other window, and its panels stay live), then
  disconnect `carefulcrab` (window returns to the picker, and `ps` shows its `ssh` forward gone).
