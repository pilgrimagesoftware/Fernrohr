# Design

## Context

The shell's `DockArea` (gpui-component) already implements split/move/close of panels by mouse
drag internally; GPUI Kit's panel/group model exposes panels grouped into tab strips, which are
themselves arranged in a split tree. `app/src/ui/nav.rs`'s `add_panel` and `app/src/shell.rs`'s
`watch_workspace` are the existing entry points that build and observe a window's `DockArea`.
Every other keyboard-first feature in the shell (see `panel-focus-navigation`,
`panel-focus-highlight-inset`) already tracks which panel/group currently has focus, so this
change can read that same focus state rather than introduce a new one.

## Goals / Non-Goals

**Goals:**
- Expose split/move/close-group/merge-group as commands operating on "the focused group" and
  "the focused panel", driven entirely by `DockArea`'s existing mouse-driven primitives.
- Keep the new commands indistinguishable in outcome from the equivalent drag gesture, so no new
  persisted-layout format or restore-on-launch logic is needed.

**Non-Goals:**
- No new layout persistence work; `app-shell`'s existing workspace persistence already
  serializes whatever arrangement `DockArea` ends up in, split/move/close by keyboard or mouse.
- No change to how a panel's own content keybindings (e.g. a resource-browser panel's filter
  keybinding) are scoped; this only adds group/workspace-level actions.
- No "pick a direction with the mouse" UI; direction is always a parameter baked into the action
  (e.g. four separate split actions, one per direction) so each has its own default keybinding.

## Decisions

- **One action per direction, not one parameterized action.** GPUI's `actions!` macro and keymap
  format bind one key combination to one action id. `SplitGroupLeft`, `SplitGroupRight`,
  `SplitGroupUp`, `SplitGroupDown` (and the `Move`/`Merge` equivalents) each get their own
  default binding, consistent with how other directional actions already exist in the command
  registry. Alternative considered: a single `SplitGroup(Direction)` action with the direction
  read from a palette prompt — rejected because it adds a modal step to an operation that should
  be instant from the keyboard.
- **"Adjacent group in a direction" is resolved geometrically**, using the focused group's
  rendered bounds and picking the nearest group whose bounds are entirely on that side —
  the same notion of adjacency `panel-focus-navigation`'s directional focus movement already
  uses, so the two features agree on what "the group to the right" means.
- **Closing a group reuses the single-panel close path per panel**, iterating the group's panels
  and invoking the same confirmation-aware close used by `ClosePanel` today, rather than adding a
  bulk-close code path with its own confirmation logic. This keeps the tunnel-aware confirmation
  behavior in one place.
- **Context predicate: "a panel group has focus."** These actions are unavailable (and absent
  from the palette's enabled list, though still visible as disabled/searchable) when focus is
  outside the dock area entirely (e.g. a modal dialog is open), matching the existing pattern for
  other dock-scoped commands.

## Risks / Trade-offs

- [Geometric adjacency can be ambiguous in an irregular split tree (e.g. an L-shaped layout)] →
  Mitigate by falling back to "no adjacent group" (a no-op, per the specified scenario) rather
  than guessing; users can always fall back to the mouse for unusual layouts.
- [Closing a group with multiple confirmable panels could show several confirmation dialogs in a
  row] → Mitigate by batching into a single confirmation dialog listing all panels in the group
  that need confirming, so the user isn't asked once per panel.
