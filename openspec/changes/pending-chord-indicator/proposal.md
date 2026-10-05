# Proposal

## Why

`panel-move-keybindings` added the app's first two-step shortcuts (`cmd-k <arrow>`, `cmd-k w`).
After the first key, nothing on screen shows that the app is waiting for a second key or which keys
would complete the shortcut. A user who pressed `cmd-k` by accident, or who forgot the second key,
has no feedback. When the keys typed so far are also a complete shortcut, the toolkit gives up
waiting after a fixed one second, which is too short for anyone still looking up the next key.

## What Changes

- Show the pending keys in the status bar, such as `⌘K …`, while a two-step shortcut waits for its
  next key. The indicator clears as soon as the shortcut completes, is abandoned, or focus moves.
- Next to the indicator, list the keys that can complete the pending shortcut in the current
  context, each with its command's title, sourced from the command registry and the effective
  keymap.
- Add a "Shortcut timeout" preference, 3 seconds by default and adjustable from 1 to 10 seconds,
  in `ui.toml` and the Settings panel. It sets how long the app waits for the next key when the
  keys so far are also a complete shortcut. A pending shortcut that is not itself a complete
  shortcut keeps waiting with no timeout, as it does today.

## Capabilities

### New Capabilities

(none)

### Modified Capabilities

- `command-system`: adds a visible indicator for multi-step shortcuts that are waiting for their
  next key, with the keys that can complete them, and a user-adjustable timeout for the ambiguous
  case.

## Impact

- `App/app/src/ui/status_bar.rs`: the pending-keys indicator and the completion list.
- `App/app/src/keymap.rs` / command registry: a query for the bindings that extend a given
  keystroke prefix in the focused context.
- `App/app/src/config/ui.rs` and `App/app/src/ui/settings/`: the `shortcut_timeout_secs`
  preference and its Settings row.
- Uses GPUI's existing public pending-input API (`observe_pending_input`, `pending_input`,
  `set_pending_input_timeout_paused`). No dependency changes.
