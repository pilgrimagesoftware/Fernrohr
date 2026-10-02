# Tasks

## 1. Init Container Status Display

- [ ] 1.1 Investigate Pod detail rendering for init containers and identify where waiting state is misrepresented. Verify by inspecting the relevant component that renders container statuses.
- [ ] 1.2 Fix init container waiting state rendering to show "Waiting" with reason/message clearly. Verify by viewing a Pod with a waiting init container - the UI shows waiting status explicitly.

## 2. YAML View Enhancements

- [ ] 2.1 Add vertical scrolling to the YAML view in resource detail panes. Verify the YAML content scrolls vertically when content exceeds the viewport height.
- [ ] 2.2 Add folding/collapsing support for nested YAML structures in the detail pane YAML view. Verify nested nodes can be folded and unfolded with keyboard and mouse interactions.

## 3. Copy Resource Name Action

- [ ] 3.1 Register a new command/action `copy_resource_name` in the command registry with title, default keybinding, and appropriate context predicate. Verify the action appears in the command palette when a resource is selected.
- [ ] 3.2 Wire the action to copy the selected resource's name to the system clipboard. Verify invoking the action copies the correct resource name.
- [ ] 3.3 Add the action to relevant menus/keybindings per keyboard-first conventions. Verify the action is accessible via keyboard.

## 4. Per-Field Copy Affordances

- [ ] 4.1 Add hover-activated copy buttons for Pod container image fields in the detail pane. Verify buttons appear on hover and copy the image value to clipboard.
- [ ] 4.2 Add hover-activated copy buttons for ConfigMap key and value fields. Verify buttons appear on hover and correctly copy key and value.
- [ ] 4.3 Ensure copy buttons are keyboard-focusable and activatable via keyboard (Enter/Space). Verify keyboard navigation and activation works per keyboard-first requirements.
- [ ] 4.4 Apply the same copy affordance pattern to other obvious fields where beneficial (if discovered). Verify consistent behavior.

## 5. Status Colorization

- [ ] 5.1 Add semantic styling for status values (Running/Succeeded, Pending/Waiting, Failed/Error/Terminated). Verify styles follow the existing theme.
- [ ] 5.2 Apply colorization to Pod status in the detail pane. Verify Pod status displays with appropriate colors based on state.
- [ ] 5.3 Extend colorization to other resource statuses where applicable. Verify no regressions and accessibility (sufficient contrast, text labels remain clear).

## 6. Managed Fields Disclosure Control

- [ ] 6.1 Replace the flat managed fields rendering with a collapsible disclosure control (expand/collapse), mirroring ConfigMap key/value presentation. Verify managed fields can be expanded/collapsed.
- [ ] 6.2 Set a sensible default state (collapsed for large managed fields) to reduce visual noise. Verify the default is readable and discoverable.
- [ ] 6.3 Ensure the disclosure control is keyboard navigable and operable. Verify keyboard-first compliance.

## 7. Verification

- [ ] 7.1 Run cargo test to ensure no regressions. Verify all tests pass.
- [ ] 7.2 Run linting/type checks as per project conventions. Verify the codebase remains clean.