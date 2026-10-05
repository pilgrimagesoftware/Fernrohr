---
paths:
  - "**/*.rs"
  - "App/**/*.rs"
  - "openspec/changes/**"
---

# Icon buttons with tooltips, not text buttons

Outside dialogs, controls are icon buttons (an icon, image or symbol) with an informative tooltip,
not text-labelled buttons. Icons take less space and read the same in any language.

- **Tooltip on every icon button.** It names the action in plain words ("Stop port-forward",
  "Copy address") and, when the action is a registered command with a key, shows that key from the
  live keymap, the same way hint rows do.
- **Dialogs are the exception.** Confirmation and other dialog buttons keep text labels ("Delete",
  "Cancel") and also show their bound key (see `keyboard-first.md`).
- **Prefer the existing icon set.** Use the app's bundled icons and gpui-kit's `IconName` before
  adding new artwork, and keep one meaning per icon across the app.
- **Still keyboard-first.** An icon button is a tab stop and activates with Enter or Space, and
  its action is also a registered command. The icon is just one more way to reach it.
- **Tests find icon buttons by id or debug selector, not label text.** Assert the tooltip text as
  well, so a missing tooltip fails the test.
