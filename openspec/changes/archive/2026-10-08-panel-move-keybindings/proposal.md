# Proposal

## Why

The dock area already supports splitting, moving, and closing panels by dragging them with a
mouse, but Fernrohr is keyboard-first: every feature must be fully usable without a mouse. Panel
layout is the one part of the shell that currently has no keyboard path, which breaks that
guarantee and slows down users who rearrange panels often (e.g. splitting a cluster's pod list
next to its logs panel).

## What Changes

- Add keybindable actions to split the focused panel's group in a direction (left, right, up,
  down), opening a new split pane that keeps the focused panel's panel-opening target.
- Add a keybindable action to move the focused panel into an adjacent group (the next group over
  in a given direction), merging it into that group's tab strip.
- Add a keybindable action to close the focused panel's entire group (all panels in that group's
  tab strip at once), reusing the existing tunnel-aware confirmation path for any panel in the
  group with state that warrants confirming.
- Add a keybindable action to merge the focused panel's group into an adjacent group, collapsing
  the split.
- Register every new action in the command registry (default keybinding, title, and
  availability context limited to "a panel group has focus") so they appear in the command
  palette and the keymap file alongside existing commands.

## Capabilities

### New Capabilities

(none)

### Modified Capabilities

- `app-shell`: panels can now be split, moved between groups, and groups closed or merged via
  keyboard commands, not only by dragging.

## Impact

- `App/src/ui/` dock/workspace layer that wraps gpui-component's `DockArea` (split, move-panel,
  close-group, merge-group operations).
- `App/src/command.rs` and the central command registry (new action ids, default bindings).
- `keymap.toml` defaults (new entries) and the keybindings editor's list of editable commands.
- Existing per-panel keybinding/context-predicate pattern used by resource-browser panels.
