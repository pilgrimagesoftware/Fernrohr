# Proposal

## Why

The app has no application menu at all - no App/File/Edit/View/Navigate/Window/Help bar - so
nothing is discoverable except by already knowing the command palette exists and what to type
into it. A user opening the app cold has no way to find "switch cluster," "close panel," or any
other action short of guessing a keybinding. Separately, the UI and monospace surfaces (YAML
detail, logs) render with placeholder font families rather than the app's actual chosen
typefaces, so text rendering doesn't match the intended design.

## What Changes

- Add a native application menu bar with seven top-level menus in this order: App, Context (the
  "File" menu - opens/switches cluster contexts), Edit, View, Navigate, Window, Help.
- Populate each menu's items by reading from the existing `CommandRegistry` rather than
  duplicating action ids/titles/keybindings - a menu item and its palette/keymap entry are the
  same registered command, always in sync.
- Menu items that have no sensible corresponding command yet (e.g. platform-standard Edit items
  when there's no text editing surface) are omitted rather than stubbed.
- Wire the two concrete fonts the design calls for: Manrope for UI text, Monaco for monospace/
  terminal contexts (YAML detail, logs), replacing the current placeholder mono font family.

## Capabilities

### New Capabilities
- `application-menu`: the app's native menu bar, its fixed top-level structure, and how its
  items are sourced from the command registry.
- `typography`: the app's chosen UI and monospace font families and where each applies.

### Modified Capabilities
(none - no existing capability's requirements change)

## Impact

- `app/src/main.rs`: menu construction added to app startup, alongside existing `theme::init`.
- `crate::command::CommandRegistry`: read (not modified) to build menu items; may need a
  `category`/`menu` grouping field if none already distinguishes which top-level menu an action
  belongs under.
- `crate::ui::theme`: font family fields wired to `Theme`'s existing UI/mono font slots.
- No changes to persisted config formats or existing window/dock behavior.
