# Tasks

## 1. Locate and Understand Current Top Bar Implementation

- [ ] 1.1 Find the main window top bar implementation in the App codebase. Verify the file(s) and component structure.
- [ ] 1.2 Identify how connected contexts are rendered and how the add context action/button is implemented. Verify current behavior.

## 2. Integrate gpui-kit Toolbar

- [ ] 2.1 Import/prepare to use gpui-kit Toolbar component in the main window. Verify correct import paths per codebase conventions.
- [ ] 2.2 Replace the current top bar container with gpui-kit Toolbar. Verify the toolbar renders without layout errors.

## 3. Reconstruct Toolbar Layout

- [ ] 3.1 Add icon to the Toolbar in the correct position. Verify icon displays as before.
- [ ] 3.2 Add app name to the Toolbar. Verify app name displays correctly.
- [ ] 3.3 Move the connected contexts list into the Toolbar after app name. Verify connected contexts display correctly and update when contexts change.
- [ ] 3.4 Add the "add context" button to the Toolbar at the end. Verify the button is visible and positioned correctly.

## 4. Preserve Behavior and Accessibility

- [ ] 4.1 Ensure the add context button retains its existing action/handler. Verify clicking and activating via keyboard still works.
- [ ] 4.2 Verify keyboard navigation and focus order remain correct (keyboard-first compliance). Verify tab order is logical.
- [ ] 4.3 Test connected contexts updates when connections change state. Verify dynamic updates still work.

## 5. Styling and Polish

- [ ] 5.1 Apply appropriate spacing/alignment within Toolbar to match the intended layout. Verify visual consistency.
- [ ] 5.2 Ensure no visual regressions compared to previous top bar. Verify styling integrates with gpui-kit theme.

## 6. Verification

- [ ] 6.1 Run cargo test to ensure no regressions. Verify all tests pass.
- [ ] 6.2 Run linting/type checks as per project conventions. Verify the codebase remains clean.