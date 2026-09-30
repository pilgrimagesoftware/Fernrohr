# Tasks

The App tasks here ship together with `per-tab-close-button`'s tasks 2.x as one App task,
`tab-keyboard-app`. It starts after Builder 2's split-shell merges.

## 1. Focused tab group and tab stepping

- [ ] 1.1 Add the focused-group helper next to `ui/panel/focus.rs`'s `dock_stops`. It returns the
  group whose displayed panel `contains_focused`, else the first stop's group, else `None` for an
  empty dock, as the group's `panels` and `active_ix`. Verify with tests over a real `DockSkin`
  dock: focus inside a group, focus outside the dock (the fallback), and an empty dock.
- [ ] 1.2 Add tab targeting. Next/previous reuse `focus::step` over the group's panels.
  "Select N" indexes 1..8 directly, 9 means the last tab, and a position past the end gives none.
  Verify with unit tests: wrap at both ends, one tab, select 3 of 5, select 9 of 4, select 5 of 2.

## 2. Tab commands

- [ ] 2.1 Register `tab.next` ("Next Tab", `cmd-shift-]`) and `tab.previous` ("Previous Tab",
  `cmd-shift-[`) in `MenuSlot::Navigate`, and `tab.select_1`…`tab.select_9` ("Select Tab N",
  `ctrl-1`…`ctrl-9`, no menu slot). All global. Verify: a test that `registry.available(&[])`
  lists all eleven, and the registry-wide test that no two commands share a default binding.
- [ ] 2.2 Handle them on `MainWindow`, in the post-split-shell module layout. Find the target,
  `select_panel` it, and focus its handle. Verify with real-keystroke tests:
  - with Pods and a pod detail as tabs in one group, `cmd-shift-]` shows and focuses the detail
    and wraps back;
  - `ctrl-2` selects the second tab;
  - `ctrl-9` selects the last tab;
  - from the Resource panel, `cmd-shift-]` acts on the first group;
  - from the Resource filter field, the keys still reach the commands.

## 3. Cmd-W on the focused tab group (`per-tab-close-button` 2.x)

- [ ] 3.1 Implement `per-tab-close-button` tasks 2.1–2.6 with step 1 as noted in its design: when
  there's a focused group, focus its displayed tab, then dispatch `ClosePanel`; otherwise take
  the close-window path with its tunnel confirmation. Verify with those tasks' keyboard tests,
  plus one more: `Cmd-W` with focus in the Resource panel closes the first group's displayed tab,
  and the window stays open.

## 4. Verification

- [ ] 4.1 `cargo fmt -- --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` all
  pass, and new or touched files stay under the 500-line limit.
- [ ] 4.2 Manual check on a running build. Open several pod detail tabs, then cycle them with
  `cmd-shift-]` / `cmd-shift-[` and jump with `ctrl-1`…`ctrl-9`, confirming the underline follows.
  Press `Cmd-W` from inside a tab and from the Resource panel, then with an empty dock (the
  window closes, or confirms first if a tunnel is active). **Needs user confirmation.**
