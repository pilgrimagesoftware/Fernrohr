# Design

## Context

Closing a focused panel tab currently causes focus to disappear; clicking another tab afterward doesn't properly grant focus. This involves focus management in the dock/tab UI (likely using gpui-component Dock or the panel tab implementation). The fix needs to ensure focus is transferred on close and that tab activation sets focus correctly.

See proposal.md for motivation; requirements in window-management spec delta.

## Goals / Non-Goals

**Goals:**
- Ensure closing a focused tab transfers focus to a sensible target (adjacent tab if exists, otherwise parent dock/container).
- Ensure clicking/activating a tab sets keyboard focus to the associated panel content.
- Preserve existing tab close/activation behavior otherwise.

**Non-Goals:**
- Changing tab ordering or dock layout semantics.
- Modifying tab styling.

## Decisions

1. **Focus transfer on tab close**: In the tab close handler, if the closed tab had focus, determine the next focus target before or immediately after removal: prefer the right neighbor, else left neighbor, else focus the dock/panel area. Use GPUI's focus management APIs to explicitly focus the target.

2. **Tab activation sets focus**: When a tab is activated (via click, keyboard, or programmatically), ensure the tab view and its associated panel content receive focus. Verify the focus is set on the correct element so keyboard input works immediately.

3. **Consistent with keyboard-first**: Ensure the fix works for both mouse clicks and keyboard-driven tab operations (close/next/previous tab). Follow existing focus handling patterns in the codebase.

## Risks / Trade-offs

- [Risk] Choosing wrong fallback focus target could break workflow → Mitigation: Prefer adjacent tab (most intuitive), fallback to dock area as conservative choice.
- [Risk] Timing issues if focus is set before removal completes → Mitigation: Compute target before removing the tab, or set focus after ensuring the target still exists.
- [Risk] May affect other focus flows in dock → Mitigation: Test tab close scenarios (last tab, middle tab, first tab).