# Design

## Context

See `proposal.md` for why. Constraints from the code and from GPUI (`gpui-pre` 0.3.6,
gpui-component 0.6.6):

- **The keymap isn't kept after startup.** `keymap::load` → `keymap::bindings` →
  `cx.bind_keys(..)` runs once in `shell::init`, and the `KeymapConfig` is dropped after that. The
  `CommandRegistry` is kept as a global.
- **GPUI rebinds at runtime.** `bind_keys` and `clear_key_bindings` both queue a refresh of every
  window, and `Kbd::binding_for_action` (hint rows, the palette) reads the live keymap, so those
  update on the next frame. Later bindings win, and `Unbind(action_name)` removes an action's
  earlier binding for a keystroke. `clear_key_bindings` is unusable here: it would also drop
  gpui-component's own bindings (text input, lists, the dock) and the Resource panel's raw ↑/↓.
- **The menu bar is built once.** `menu::init` calls `cx.set_menus` once, and it also registers
  the menu actions' handlers. Rebuilding the menu means calling `set_menus` again without
  re-registering those handlers.
- **Where keystrokes reach first.** `App::intercept_keystrokes` sees each `KeyDownEvent` before
  binding lookup; stopping propagation there keeps the bound action from running. `on_key_down`
  runs only after action dispatch, too late to record `cmd-w`. Modifier-only presses never reach
  interceptors.
- **No key-recorder widget** exists in gpui-component. `Kbd` only displays keys.
- **No predicate overlap test.** GPUI has no general "can these two binding contexts both be
  active" check. The registry's contexts are single key-context names or global.
- **A pattern to follow:** `ui/tunnels/`, a single-instance window (`open_or_focus`, a handle
  global cleared on close) with a list and an inline editor.

## Goals / Non-Goals

**Goals:** see the spec. Plus: `keymap.toml` stays the source of truth and stays hand-editable.

**Non-Goals:**
- Recording multi-keystroke chords (`cmd-k cmd-b`) or modifier-only keys. Chords already in
  `keymap.toml` still work and are displayed; the recorder captures one keystroke.
- Other settings sections. The window is structured for more, but ships with one.
- Watching `keymap.toml` for hand edits while the app runs. Hand edits apply on relaunch, as
  today.
- Import, export, or per-window keymaps.

## Decisions

### A Settings window, `cmd-,`, App menu

`settings.open` ("Settings…", `cmd-,`, `MenuSlot::App`, global) calls `settings::open_or_focus`,
the Tunnels-window pattern. The window lists sections in a sidebar, with only Keyboard Shortcuts
for now. *Why not a "Keyboard Shortcuts" window:* `cmd-,` and the App menu's Settings item are
where every macOS app puts preferences. Giving them to a shortcuts-only window would mean moving
them when the first other setting arrives. *Why not a dock panel:* settings are app-wide, not
per-cluster, and a panel would be saved into each window's layout.

### The keymap becomes a live global

`shell::init` stores the loaded `KeymapConfig` as a global, `LiveKeymap`, together with the
keymap path. The editor edits it through three `keymap` functions:

- `set_override(id, keys)`
- `reset(id)`, which removes the entry so the default applies
- `remove(id)`, which stores an empty string so the command has no key

The empty string is `resolve`'s existing "no override" case, so it needs a small change: an
explicitly empty entry means *no key*, distinct from an absent entry, which means *default*.
Each call saves the file with `config::save` and then re-binds that one command.

*Why write overrides, not the whole map:* the file keeps its first-run shape. New entries are
added only for commands the user changed, so the file doesn't fill with defaults.

### Re-binding one command, not the whole keymap

For a change to command C, from old key O to new key N:

1. Bind `O → Unbind(C's action name)` in C's context, so O stops triggering C.
2. Bind `N → C`, unless C's key was removed.

Both go through `bind_keys`, which appends and refreshes windows. The keymap grows by two
bindings per edit. That's negligible, and relaunch starts clean from `keymap.toml`.

*Alternative:* `clear_key_bindings` then rebinding everything from the registry. That would drop
every binding the registry doesn't own (gpui-component's, the Resource panel's ↑/↓) with no way to
re-add them.

### Recording with a keystroke interceptor

While a row is recording, the recorder holds a `cx.intercept_keystrokes` subscription. The first
`KeyDownEvent` becomes the new key. `Escape` with no modifiers cancels. The interceptor stops
propagation, so nothing bound to that key runs. Dropping the subscription ends recording, whether
from a key, a click elsewhere, or the window closing.

A key with no modifier, like `d`, is allowed for panel-scoped commands (that's the Pods panel's
own style). For a global command it's recorded with a warning, because it would fire while typing
in text fields that don't handle it.

### Conflicts: same scope or global

Two commands conflict on a key when both are global, or both are in the same key context. A
global and a panel command sharing a key don't conflict: GPUI's depth rule gives the panel
binding precedence inside that panel, and the global one wins everywhere else. The editor shows
that as "Pods panel: `d` = Describe; elsewhere = …". Conflicts get a confirm: "Also used by X.
Apply anyway?" Applying leaves both bound; the later binding wins, as GPUI already resolves it.
The editor then flags both rows. Keys are compared after parsing, so equivalent spellings match.

*Why not a general overlap test:* GPUI has none, and every registry context today is a single
name or global, so exact scope comparison is complete for this registry.

### Menu labels

Split `menu::init` into `register_handlers` (once, at startup) and `rebuild_menus`, which
`set_menus` from the registry and the live keymap. Every keymap edit calls `rebuild_menus`.

### Keyboard route

The editor's actions are registered commands, in the Settings window's `KeyboardShortcuts` key
context:

| id | action | default key |
|---|---|---|
| `settings.shortcuts.record` | Record Shortcut | `enter` |
| `settings.shortcuts.reset` | Reset Shortcut to Default | `cmd-backspace` |
| `settings.shortcuts.remove` | Remove Shortcut | `backspace` |
| `settings.shortcuts.filter` | Filter Shortcuts | `/` |

↑/↓ row movement stays a raw binding, since the rule exempts pure cursor movement. Tab reaches the
filter field and the rows.

## Risks / Trade-offs

- **[A user records a key that the OS takes (`cmd-tab`, `cmd-space`).]** The app never sees the
  keystroke, so nothing is recorded, and the row stays in recording until Escape or a click. The
  recording prompt says "press a shortcut, or Escape to cancel".
- **[Global single-letter keys.]** They're allowed with a warning, since some users want them. The
  warning explains they may fire while typing.
- **[`Unbind` bindings accumulate over a session.]** It's two bindings per edit. A session with
  hundreds of edits is still well within what the keymap handles, and relaunch resets it.
- **[`remove` changes what an empty `keymap.toml` entry means.]** Today an empty entry falls back
  to the default. After this change it means no key. Nobody writes empty entries by hand today,
  since the first-run file never contains one, and the spec's "Reset and remove" scenario pins the
  new meaning.
- **[Split-shell sequencing.]** `shell::init` gains two lines (store `LiveKeymap`, register
  `settings.open`). Builder 2's split-shell is moving that code, so the App side of this change
  starts after split-shell merges.

## Open Questions

- Should the Settings sidebar list future sections as disabled placeholders, or only what
  exists? Only what exists, until there's a second section. This doesn't change the spec.
