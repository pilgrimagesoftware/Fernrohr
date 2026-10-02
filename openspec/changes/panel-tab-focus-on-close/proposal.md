# Proposal

## Why

When closing a focused panel tab, focus is lost and the next click on a panel tab doesn't properly restore focus. This breaks keyboard-first navigation and requires an extra click to regain keyboard control over the panel.

## What Changes

- **Bugfix**: Ensure focus is properly transferred when closing a focused panel tab (transfer focus to an adjacent tab, the dock area, or a sensible fallback) so focus doesn't disappear.
- **Focus restoration**: Fix the interaction so that clicking a panel tab after a close correctly gives it focus (keyboard input works immediately).

## Capabilities

### New Capabilities

None.

### Modified Capabilities

- `window-management`: Updates panel/tab close behavior to maintain keyboard focus and ensure tab activation properly grants focus to the panel.

## Impact

- Affects dock/panel tab UI in the App submodule (tab close handling, focus management).
- No API changes; purely behavioral fix for focus handling in the window/dock system.
- Improves keyboard-first workflow as specified in project conventions.