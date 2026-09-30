---
paths:
  - "**/*.rs"
  - "App/**/*.rs"
  - "openspec/changes/**"
---

# Keyboard navigation is a first-class feature

Fernrohr is keyboard-first (k9s-inspired). Every feature must be fully usable from the keyboard and
from the mouse. Keyboard support is a requirement, not polish added after the mouse version works.
A feature that only one input can reach isn't done.

## What every UI change must satisfy

- **Every action has both routes.** Anything you can do with a click (select, open, connect, edit,
  delete, confirm, cancel) also has a keyboard route, and vice versa. Write both down in the
  proposal's specs and tasks, not just the mouse one.
- **Actions go through the command registry.** A user-facing action is a `Command` (id, title,
  default binding, `KeyContext`, and `menu` slot if it belongs in the menu bar). That gives it a
  keybinding, a palette entry and a menu item at once. Don't wire a one-off click handler with no
  command behind it.
- **One selection model.** Clicks and keyboard navigation move the same selection; hover never does.
  gpui-component's `Command` selects on hover, so tell the two apart with
  `window.last_input_was_keyboard()` rather than copying its `on_select` straight into your state.
  (The cluster picker's `follow_keyboard` in `ui/picker.rs` is the reference.)
- **Show the keys.** Panels and dialogs with actions show their shortcuts in a hint row, read from the
  live keymap with `Kbd::binding_for_action` (see the Pods panel and `ui/picker_keys.rs`).
- **Dialogs are keyboard-operable.** Tab reaches every control, Enter or Space activates, Escape
  cancels, and focus goes somewhere sensible when the dialog opens and returns when it closes.
  gpui-component `Button`s are tab stops by default; don't turn that off without a replacement.
- **Focus is deliberate.** A new view, window or dialog sets its initial focus. A panel's own
  bindings live in its `KeyContext`, so they fire only while it's on the focus path.

## Tests

Cover the keyboard route, not only the mouse one: simulate real keystrokes
(`VisualTestContext::simulate_keystrokes`) so `last_input_was_keyboard()` and the actual keymap are
exercised. Calling the handler directly skips the path that breaks. A test that only calls
`on_click` handlers doesn't show a feature is keyboard-reachable.
