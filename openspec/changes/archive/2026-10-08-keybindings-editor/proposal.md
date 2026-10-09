# Proposal

## Why

Every key binding in Fernrohr is meant to be user-configurable, and technically it is:
`keymap.toml` overrides any command's key by its id. In practice, you can't find what to
configure. The file is written once, on first run, and never updated, so an existing user's file
lists only the commands that existed then (today, four of about forty). Every command added since
(the Pods, pod detail, Resource and panel-focus shortcuts) works if you add it by hand, but you'd
have to know its id. Changes also need a restart, and nothing warns when two commands end up on
one key. The user picked an in-app editor over patching the file at startup.

## What Changes

- **A Settings window with a Keyboard Shortcuts section.** It's opened by a new registered command,
  `settings.open` ("Settings…", default `cmd-,`, in the App menu). That's the first command in
  `MenuSlot::App`, which the registry has reserved for this. Keyboard Shortcuts is the only
  section for now; later settings join it rather than each getting a window.
- **Every registered command, listed.** Each row shows its title, where it applies (Global, or a
  panel such as "Pods panel"), and its current key, marked when it differs from the default. The
  list is searchable by title, id or key. Palette-only commands (no default key) are listed too,
  and can be given one.
- **Recording a key:** choose a row, press the new key combination, and it's captured before
  any existing binding can fire. So recording `cmd-w` records it rather than closing the window.
  Escape cancels. Each row can also be reset to its default, or have its key removed.
- **Conflict warnings:** when the recorded key is already another command's in an overlapping
  scope (both global, or the same panel), the editor names that command before applying. When a
  panel command shadows a global key inside that panel, the editor says so.
- **Changes apply immediately:** they're saved to `keymap.toml` and re-bound live, so there's no
  restart. Key hints and palette labels update on the next frame, and the menu bar's shortcut
  labels are rebuilt. The file stays hand-editable, with the same format and ids.
- **Keyboard-first:** every editor action is a registered command in the editor's key context,
  with keyboard routes: ↑/↓ rows, Enter to record, `/` to filter, reset and remove.

## Capabilities

### New Capabilities
(none)

### Modified Capabilities
- `command-system`: gains an in-app editor for the user-editable keymap. The existing keymap
  requirement is unchanged; this adds one beside it.

## Impact

- New `app/src/ui/settings/` (window, Keyboard Shortcuts section, key recorder), modelled on
  `ui/tunnels/` (single-instance window, list with an inline editor).
- `app/src/keymap.rs`: save one override, reset or remove it, and re-bind a single command live.
  It never clears the whole keymap, because that would also drop gpui-component's own bindings.
  Also a conflict query over the registry.
- `app/src/ui/menu.rs`: split menu-bar building from action-handler registration, so the menu can
  be rebuilt after a rebind without registering handlers twice.
- `app/src/command.rs`: `MenuSlot::App` gets its first command, and its `UNWIRED` marker goes.
- No new dependencies.
