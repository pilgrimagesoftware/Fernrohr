# Tasks

The App side starts after Builder 2's split-shell merges (`shell::init` is moving).

## 1. A live, editable keymap

- [ ] 1.1 Keep the loaded `KeymapConfig` and its path as a `LiveKeymap` global, set in `init`.
  Verify with a test that after `init` the global holds the file's overrides.
- [ ] 1.2 `resolve`: an explicitly empty entry means "no key", and an absent entry means
  "default". Verify with unit tests for both, plus the existing invalid-entry fallback.
- [ ] 1.3 `keymap::set_override` / `reset` / `remove`: each updates `LiveKeymap`, saves with
  `config::save`, and re-binds one command. That's `Unbind(action)` on the old key in the
  command's context, then the new binding, never `clear_key_bindings`. Verify with tests. After
  each call, the saved file round-trips. Resolving the old key no longer yields the command, and
  the new key does. A gpui-component binding (the text input's) still resolves.
- [ ] 1.4 `keymap::conflicts(registry, live, id, keys)`: returns commands sharing `keys` in the
  same scope (both global, or the same context), and separately the panel-over-global shadows.
  Keys are compared after parsing. Verify with unit tests: global-global conflict, same-panel
  conflict, panel-shadows-global (not a conflict), different panels (none), equivalent spellings.

## 2. Menu labels follow the keymap

- [ ] 2.1 Split `menu::init` into `register_handlers` (once) and `rebuild_menus` (callable
  repeatedly). Every keymap edit calls `rebuild_menus`. Verify: a test that after
  `set_override` the menu model's item for that command carries the new keystroke, and that menu
  actions still fire exactly once.

## 3. Settings window and Keyboard Shortcuts section

- [ ] 3.1 Register `settings.open` ("Settings…", `cmd-,`, `MenuSlot::App`, global), and remove
  `MenuSlot::App`'s `UNWIRED` marker. Add `ui/settings/` with `open_or_focus`, a single-instance
  window following `ui/tunnels/`, with a sections sidebar containing Keyboard Shortcuts. Verify
  with a keystroke test: `cmd-,` opens it, a second `cmd-,` focuses the same window, and it's in
  the App menu.
- [ ] 3.2 Keyboard Shortcuts list: every registered command with its title, scope ("Global" or
  the panel's name), current key (via `Kbd`), and a changed marker. Includes palette-only
  commands. The filter matches title, id or key. Verify with a test that every registry command
  has a row, and that filtering by a key string finds its command.
- [ ] 3.3 Key recorder: a `cx.intercept_keystrokes` subscription while recording. The first key
  becomes the recording, and Escape cancels. Propagation is stopped, and dropping the
  subscription ends recording. Verify with keystroke tests:
  - recording `cmd-w` records it and doesn't close the window;
  - Escape cancels without changing anything;
  - a recorded key works immediately.
- [ ] 3.4 Conflict confirm and shadow display, from `keymap::conflicts`. Verify: recording a
  global key already used by another global command shows the confirm, and declining changes
  nothing. A panel-over-global pair shows the shadow note.
- [ ] 3.5 Editor commands in the `KeyboardShortcuts` context: record (`enter`), reset
  (`cmd-backspace`), remove (`backspace`), filter (`/`). ↑/↓ move rows as a raw binding. Verify
  with keystroke tests for each. Removing leaves the command in the palette with no key.

## 4. Verification

- [ ] 4.1 `cargo fmt -- --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` all
  pass, and new or touched files stay under the 500-line limit.
- [ ] 4.2 Manual check on a running build. Open Settings with `cmd-,`. Rebind `panel.focus_next`
  and confirm the new key works at once and shows in the Navigate menu. Record a conflicting key
  and decline. Reset and remove. Relaunch and confirm the changes persisted in `keymap.toml`.
  **Needs user confirmation.**
