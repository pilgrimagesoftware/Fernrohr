# Tasks

## 1. Investigate Focus Behavior

- [ ] 1.1 Locate the panel tab close handling code in the dock/tab implementation. Verify where focus is managed on close.
- [ ] 1.2 Locate tab activation/click handling code. Verify how focus is set when a tab is selected. 

## 2. Fix Focus Transfer on Tab Close

- [ ] 2.1 Modify tab close logic to determine appropriate focus target before removal (adjacent tab, else dock/container). Verify logic covers edge cases (last tab, only tab, middle tab).
- [ ] 2.2 Explicitly transfer focus to the chosen target when closing a focused tab. Verify focus is not lost after close.

## 3. Fix Tab Activation Focus

- [ ] 3.1 Ensure clicking a panel tab sets keyboard focus to the tab/panel content. Verify keyboard input works immediately after clicking.
- [ ] 3.2 Ensure keyboard-driven tab activation also sets focus correctly. Verify keyboard-first behavior is preserved.

## 4. Testing

- [ ] 4.1 Test closing a focused middle tab - focus moves to adjacent tab. Verify expected behavior.
- [ ] 4.2 Test closing the last tab - focus moves to sensible fallback. Verify focus is retained somewhere usable.
- [ ] 4.3 Test the reported case: close focused tab, then click another tab - it receives focus immediately. Verify the bug is fixed.
- [ ] 4.4 Run cargo test to ensure no regressions. Verify all tests pass.
- [ ] 4.5 Run linting/type checks as per project conventions. Verify clean build.