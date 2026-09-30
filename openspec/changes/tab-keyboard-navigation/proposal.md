# Proposal

## Why

Every panel the app opens lands as a tab in the same center group, and only a group's displayed
tab is on screen. `panel-focus-navigation` moves keyboard focus between groups, but a tab hidden
behind another in the same group can't be reached from the keyboard at all. The only ways in are
clicking its tab, or knowing a command that re-opens exactly that panel, which doesn't exist for
most of them (a second pod's detail, a placeholder kind). That's a keyboard-first gap in the
default layout.

## What Changes

- **Next Tab / Previous Tab:** registered commands that switch the focused tab group to its
  next or previous tab, wrapping at the ends. The newly shown tab takes keyboard focus.
- **Select Tab 1–9:** registered commands that show the group's Nth tab, with 9 meaning the
  last tab, as in browsers. It's a small addition on the same helper.
- **All default keys:**
  - Next / Previous Tab: `cmd-shift-]` / `cmd-shift-[`, the macOS "next/previous tab" keys
    (Safari, Terminal, Xcode). These pair with the panel keys `cmd-]` / `cmd-[`: plain moves
    between panels, `shift` moves between tabs.
  - Select Tab N: `ctrl-1`…`ctrl-9`. `cmd-1`/`cmd-2`/`cmd-0` are already the app's show-panel
    keys.

  None of these collide with a registered command or a gpui-component binding. All can be rebound
  in `keymap.toml` by command id.
- **"The focused tab group"** is defined once: the group whose displayed panel keyboard focus is
  inside. With focus outside the dock (the Resource panel), it's the first group in
  panel-focus order. The same definition fixes how `per-tab-close-button`'s `Cmd-W` picks the tab
  to close.

## Relation to `per-tab-close-button`

`per-tab-close-button` (meta #27) decides that `Cmd-W` closes the active tab when the window's
dock has a panel, and otherwise closes the window. Its App tasks (2.x) ship together with this
change's, in one App task (`tab-keyboard-app`):

- **Same target:** both act on "the active tab of the focused tab group", so they share one
  helper instead of each choosing a group their own way.
- **A gap this closes:** #27's design dispatches `ClosePanel` whenever the center dock is
  non-empty. `ClosePanel` is handled by the tab group itself, so it only fires when focus is
  inside one. With focus in the Resource panel, `Cmd-W` would close nothing. Using the shared
  helper, `Cmd-W` focuses the target tab and then closes it. Its design gets a note to this
  effect.
- **Consistent keys:** closing a tab and moving between tabs use the same notion of "the current
  tab", so `cmd-shift-]` then `Cmd-W` closes exactly the tab that was just shown.

## Capabilities

### New Capabilities
(none)

### Modified Capabilities
- `app-shell`: the docked panel workspace gains keyboard switching between the tabs of a group.

## Impact

- `app/src/ui/panel/focus.rs` (or a sibling `tabs.rs`): the focused-group helper, and tab
  stepping and selection.
- `MainWindow` handlers, in the module layout Builder 2's split-shell leaves behind. The App side
  waits for split-shell.
- `openspec/changes/per-tab-close-button/design.md`: a note on the shared focused-group helper.
- `openspec/specs/app-shell/spec.md`: a requirement for tab switching.
