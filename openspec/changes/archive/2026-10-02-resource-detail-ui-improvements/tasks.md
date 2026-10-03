# Tasks

## 1. Init Container Status Display

- [x] 1.1 Investigate Pod detail rendering for init containers and identify where waiting state is misrepresented. Verify by inspecting the relevant component that renders container statuses.
- [x] 1.2 Fix init container waiting state rendering to show "Waiting" with reason/message clearly. Verify by viewing a Pod with a waiting init container - the UI shows waiting status explicitly.

## 2. YAML View Enhancements

- [x] 2.1 Add vertical scrolling to the YAML view in resource detail panes. Verify the YAML content scrolls vertically when content exceeds the viewport height.
- [x] 2.2 Add folding/collapsing support for nested YAML structures in the detail pane YAML view. Verify nested nodes can be folded and unfolded with keyboard and mouse interactions.

## 3. Copy Resource Name Action

- [x] 3.1 Register a new command/action `copy_resource_name` in the command registry with title, default keybinding, and appropriate context predicate. Verify the action appears in the command palette when a resource is selected.
- [x] 3.2 Wire the action to copy the selected resource's name to the system clipboard. Verify invoking the action copies the correct resource name.
- [x] 3.3 Add the action to relevant menus/keybindings per keyboard-first conventions. Verify the action is accessible via keyboard.

## 4. Per-Field Copy Affordances

- [x] 4.1 Add hover-activated copy buttons for Pod container image fields in the detail pane. Verify buttons appear on hover and copy the image value to clipboard.
- [x] 4.2 Add hover-activated copy buttons for ConfigMap key and value fields. Verify buttons appear on hover and correctly copy key and value.
- [x] 4.3 Ensure copy buttons are keyboard-focusable and activatable via keyboard (Enter/Space). Verify keyboard navigation and activation works per keyboard-first requirements.
- [x] 4.4 Apply the same copy affordance pattern to other obvious fields where beneficial (if discovered). Verify consistent behavior.

## 5. Status Colorization

- [x] 5.1 Add semantic styling for status values (Running/Succeeded, Pending/Waiting, Failed/Error/Terminated). Verify styles follow the existing theme.
- [x] 5.2 Apply colorization to Pod status in the detail pane. Verify Pod status displays with appropriate colors based on state.
- [x] 5.3 Extend colorization to other resource statuses where applicable. Verify no regressions and accessibility (sufficient contrast, text labels remain clear).
  Moved to follow-up issue #101 (2026-10-02); the spec only requires Pod statuses.

## 6. Managed Fields Disclosure Control

- [x] 6.1 Replace the flat managed fields rendering with a collapsible disclosure control (expand/collapse), mirroring ConfigMap key/value presentation. Verify managed fields can be expanded/collapsed.
- [x] 6.2 Set a sensible default state (collapsed for large managed fields) to reduce visual noise. Verify the default is readable and discoverable.
- [x] 6.3 Ensure the disclosure control is keyboard navigable and operable. Verify keyboard-first compliance.

## 7. Verification

- [x] 7.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` pass.
- [x] 7.2 Manual check: a pod with a waiting init container, a crashing container's color, folding a large YAML, copying a name and an image, expanding one managed-fields row.
  Passed 2026-10-02 (user).
