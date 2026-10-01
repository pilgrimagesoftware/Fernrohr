# Tasks

## 1. Window context model

- [x] 1.1 Replace `WindowMode::Workspace`'s `context_name` with `contexts: Vec<String>` and
  `active: usize`, with `connection_count` derived from `contexts.len()`, and no behavior change.
  Verify: the existing shell, picker, and restore tests pass unchanged.
- [x] 1.2 Add `ClusterRegistry::hold`/`release` keyed by window, removing the session on the last
  release. Verify: tests for two windows holding one context (one session), one releasing (session
  kept, watches live), and the last releasing (session gone and its tunnel's `ForwardRegistry` entry
  released, using a fake forward).
- [x] 1.3 Take a hold when a window enters a workspace or restores, and release on window close.
  Verify: a GPUI test opens two windows on one context, closes one, and asserts the session remains;
  closing the other removes it.

## 2. Persistence

- [x] 2.1 Add `contexts` to `WindowLayout` with a serde default, derived from saved panels'
  `cluster_context` when absent, and restore every listed context. Keep per-context dock layouts only
  for single-context windows. Verify: round-trip tests for a two-context window, a legacy file with
  no `contexts`, and a window whose second context has no panels.

- [x] 2.2 Save the live window's `contexts` in `WindowLayout`, and save a multi-context window's dock
  arrangement under an order-insensitive composite key in `dock_layouts.json` (single-context windows
  keep their context-name key). Verify: round-trip tests for a single-context window (unchanged) and a
  two-context window restored with both contexts held and its arrangement key used.

## 3. Context bar

- [x] 3.1 Add `ui/context_bar.rs` rendered under the title bar in workspace mode, with one chip per
  context showing its name, bound tunnel name, and health dot (via `ClusterRegistry::health` and
  `severity` if `connection-status-bar` has landed, otherwise no dot). A chip click sets `active`,
  kept in sync both ways with the Resource panel's cluster dropdown. Verify: GPUI tests for chip
  labels, and for a chip click and a dropdown change each updating the other.
- [x] 3.2 Add the "+" popover, wrapping the picker filtered to unused contexts. Choosing one takes a
  hold, connects or reuses the session, opens a Pods panel, and sets `active`. A failed connect shows
  on the chip and opens no panel. Verify: GPUI tests for filtering, for adding a context already
  connected in another window (no second session), and for the failure path with a fake connection.
- [x] 3.3 Add Disconnect to the chip menu, with a confirmation stating the panel count and the other
  windows' use; on confirm, close that context's panels, release the hold, and fall back to the
  picker when none remain. Verify: GPUI tests for confirm, cancel, the "stays connected in 1 other
  window" text, and last-context-to-picker.

## 4. Status bar alignment

- [x] 4.1 If `connection-status-bar` has landed, point its item source at the window's `contexts`.
  Verify: its GPUI tests pass against a two-context window, and Disconnect removes the item.

## 5. Full verification

- [x] 5.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` all
  pass.
- [x] 5.2 Manual check against QA: in one window add `greedygoat` and `carefulcrab`, arrange panels for
  both, quit and relaunch (both restored), open a second window with `greedygoat`, disconnect it from
  the first window (confirmation mentions the other window, and its panels stay live), then
  disconnect `carefulcrab` (window returns to the picker, and `ps` shows its `ssh` forward gone).

## 6. Fixes found during review (landed with this change)

Recorded here because manual review against the QA bastions turned them up; most would bite
any multi-context window, and several were bugs that predate this change.

- Re-entrant updates: a chip click or the Resource dropdown re-synced the view it came from
  mid-update (a GPUI "already being updated" panic on the first click). The sync is deferred.
- No dialog could appear in a main window: `MainWindow` never drew `Root`'s dialog, sheet and
  notification layers, so "+", Disconnect, `context.set_tunnel` and the command palette opened
  nothing visible. The Disconnect dialog also needed an explicit footer to show its buttons.
- Pod-scoped panels used the window's active context instead of the pod's: `PodSelection`
  now carries `context_name`, Logs ignores other contexts' selections, and a 404 reads as
  "Pod ... not found in namespace ... on ...", selectable and copyable (verbatim through markdown).
- `ssh` forwards outlived the app: GPUI's quit never drops `ClusterRegistry`, and the startup
  sweep was unwired. Quit now kills this run's forwards and startup sweeps orphans, verified
  against real leftovers. Kill shell-outs pass `--` (the Linux CI fix from App #22).
- Closed windows reappeared at every launch: only the last main window's layout is kept now.
- Only five commands were ever bound; every registry command is now bound from the registry
  (Cmd-Shift-T had no key). The App and Window menus gained the standard macOS items and
  shortcuts (Cmd-Q, Cmd-H, Option-Cmd-H, Cmd-M, Cmd-W).
- Picker: one selection for mouse and keyboard (click or arrows, never hover), connect by
  double-click, Connect or Enter, `context.set_tunnel` works on the selected row, keys shown
  once each, long names ellipsized so the tunnel selector stays visible.
- UI polish: capsule chips; the Resource header drawn outside `Sidebar`'s slot (no clipping);
  a resizable, remembered Resource panel width; the context shown in panel content (Pods
  "Context:" line; Pod detail and Logs headings with "(context)" only in multi-context
  windows), never in tab titles; a centered, modest Tunnels window.

