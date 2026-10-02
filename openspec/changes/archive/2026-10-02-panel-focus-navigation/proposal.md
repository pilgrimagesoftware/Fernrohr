# Proposal

## Why

Keyboard focus can't move from one dock panel to another. The dock has only `ToggleZoom` and
`ClosePanel`, and opening a panel doesn't focus it, so the only way to reach a different panel is
to click it. That contradicts the keyboard-first rule ("a feature that only one input can reach
isn't done"). It is also the open follow-up 4.1 of `panel-focus-highlight-inset`, whose tab
underline now shows *which* panel has focus but gives the keyboard no way to change it.

## What Changes

- **Focus next / previous panel:** two registered commands that move keyboard focus to the next or
  previous visible panel in the window, in a stable layout order, wrapping at the ends. Being
  registered commands, they get palette entries, default key bindings that `keymap.toml` can
  override, and menu items.
- **An opened panel takes focus:** a panel opened from any route (Resource panel row, a
  command, a Pods panel shortcut such as `d`, `l` or `y`) receives keyboard focus, so its own
  shortcuts work straight away and its tab is underlined. Re-showing an already-open panel
  focuses it the same way.
- Panels in a collapsed dock, and panels hidden behind another zoomed panel, are skipped: focus
  never lands somewhere the user can't see.

## Capabilities

### New Capabilities
(none)

### Modified Capabilities
- `app-shell`: the docked panel workspace gains keyboard focus navigation between panels, and
  opening a panel focuses it.

## Impact

- `app/src/util/shell.rs` or a new `app/src/ui/panel/focus.rs`: the next/previous commands and
  the panel ordering they walk.
- `app/src/ui/nav.rs`: opening (or re-showing) a panel focuses it.
- `app/src/command.rs` registrations and the Navigate or Window menu.
- `openspec/specs/app-shell/spec.md`: a new requirement for panel focus navigation.
