# Proposal

## Why

When closing a focused panel tab, focus is lost and the next click on a panel tab doesn't properly restore focus. This breaks keyboard-first navigation and requires an extra click to regain keyboard control over the panel.

## What Changes

- **Bugfix**: Ensure focus is properly transferred when closing a focused panel tab (transfer focus to an adjacent tab, the dock area, or a sensible fallback) so focus doesn't disappear.
- **Focus restoration**: Fix the interaction so that clicking a panel tab after a close correctly gives it focus (keyboard input works immediately).
- **Focus on activation**: Some tabs don't accept focus until the user switches to another tab and back (reported 2026-10-02 smoke test). Every way a tab becomes active - click, keyboard, newly opened, restored - SHALL focus its panel.

## Capabilities

### New Capabilities

None.

### Modified Capabilities

- `app-shell`: adds a requirement that the active panel tab always holds keyboard focus (there is no `window-management` spec; this is where docked-panel behavior lives).

## Impact

- Affects dock/panel tab UI in the App submodule (tab close handling, focus management).
- No API changes; purely behavioral fix for focus handling in the window/dock system.
- Improves keyboard-first workflow as specified in project conventions.