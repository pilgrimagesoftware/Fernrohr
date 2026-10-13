# Proposal

## Why

The typed label Logs panel (Open Logs by Label…) searches one cluster context, and the user can't
see or change which one. It inherits the context of the list it was opened from, else the window's
active one. In a window holding several contexts, that guess was wrong in Fernrohr#213: the panel
searched a context that didn't have the pod the selector was copied from, and said "No pods match".
App#268 fixed the guess. This change lets the user choose the context instead, the way the
Resource panel already offers a cluster dropdown when its window holds more than one connection.

## What Changes

- The typed panel's selector bar shows the context it searches, beside its namespace picker, when
  its window holds more than one context. With one context, nothing new is shown.
- The user can switch the panel to another of the window's contexts from that control by mouse, or
  by keyboard through a new panel-scoped command with a default key. The command is in the command
  palette and can be rebound in `keymap.toml`.
- Switching context stops the panel's streams and follows the new context's pods with the same
  selector and container filter. The namespace scope resets to all namespaces, because namespace
  names don't carry over between clusters.
- There is still one typed panel per context. Choosing a context that already has a typed panel
  focuses that panel instead of retargeting this one.
- The context the panel opens on is unchanged from App#268: the focused list's context, else the
  window's active one.
- A panel switched to another context is saved in that context, so a saved layout restores it
  there. A saved panel whose context the window doesn't hold becomes a placeholder, as for any
  other saved panel.
- Workload Logs panels (a Deployment's pods, for example) keep their fixed context and show no
  context control.

## Capabilities

### New Capabilities

(none)

### Modified Capabilities

- `selector-logs`: adds requirements that the typed panel's context is visible and can be changed,
  that switching it follows the new context's pods, and that a switched panel is saved in its new
  context. `selector-logs` is introduced by the still-unarchived `selector-logs-panel` change, so
  this change's delta adds requirements to it and must be archived after `selector-logs-panel`.

## Impact

App-only (`App` submodule), no new dependencies, no backend or API changes:

- `app/src/util/logs/labels/` - a context control in the typed panel's bar (beside `scope.rs`'s
  namespace picker), and retargeting a panel to another context: unsubscribe from the old context's
  shared Pods watch, subscribe to the new one's, and reset the namespace scope.
- `app/src/util/logs/commands.rs` - the new panel-scoped command and its default key.
- `app/src/util/shell/` - re-keying the open panel under its new context (as `rescope` already does
  for namespaces), and focusing the existing typed panel when the chosen context already has one.
- Saved layout state already records the panel's context; it must follow a switch.
- Docs: `docs/site/latest/logs-shells.md` gains a section on Open Logs by Label covering the
  selector, namespace scope and the new context control. `docs/site/latest/reference/keybindings.md`
  lists the new command's key.
