# Tasks

## 1. Investigate Focus Behavior

- [x] 1.1 Locate the panel tab close handling code in the dock/tab implementation. Verify where focus is managed on close.
  Nowhere: cmd-w (`util/shell/tabs.rs`) dispatches gpui-kit's `ClosePanel`, and the dock removes the panel
  without moving focus, which leaves `window.focused()` == None.
- [x] 1.2 Locate tab activation/click handling code. Verify how focus is set when a tab is selected.
  A tab click runs gpui-base 0.7.0's `TabGroup::select_tab`, which returns early for the tab that's
  already active, before `focus_active_panel` (upstream). Our own routes (`open_target_in`, `show_tab`)
  focus explicitly.

## 2. Fix Focus Transfer on Tab Close

- [x] 2.1 Modify tab close logic to determine appropriate focus target before removal (adjacent tab, else dock/container). Verify logic covers edge cases (last tab, only tab, middle tab).
- [x] 2.2 Explicitly transfer focus to the chosen target when closing a focused tab. Verify focus is not lost after close.
  App#85: `MainWindow::keep_focus_on_a_panel` runs on every `DockEvent::LayoutChanged`. If a closed panel
  held focus, focus moves to the tab now displayed in its group (each `OpenPanel` remembers its last
  group), else to the first open panel, else to the Resource panel or the window.

## 3. Fix Tab Activation Focus

- [x] 3.1 Ensure clicking a panel tab sets keyboard focus to the tab/panel content. Verify keyboard input works immediately after clicking.
  App#85: `panel_title::title_element` focuses its panel on mouse-down, working around the upstream
  early return, with a comment citing it at that site. Verified by
  `util::shell::tab_focus::tests::clicking_the_already_active_tab_focuses_its_panel`.
- [x] 3.2 Ensure keyboard-driven tab activation also sets focus correctly. Verify keyboard-first behavior is preserved.
  Already true: `show_tab` focuses the tab it shows. Covered by
  `util::shell::tabs::tests::shifted_brackets_cycle_the_focused_groups_tabs`.
- [x] 3.3 Fix tabs that only accept focus after switching to another tab and back: find which panels' focus handles aren't focused on first activation (newly opened, restored from layout, first click), and focus on every activation path. Verify with a test that opens a panel and, without any tab switch, dispatches a panel-scoped keystroke that reaches it - for each panel type (Pods, object list, object detail, pod detail, logs).
  Every panel type's `Focusable` returns its own tracked handle, and every open route already focused
  the new panel. The gaps were the upstream active-tab click (3.1) and a new, restored or just-connected
  workspace focusing only the window root, which App#85 fixes by focusing the displayed panel on entry.
  Verified by `util::shell::tab_focus::open_tests` (Pods `d`, pod detail `y`, object list `/`, object
  detail `y`, plus the launch and connect cases). Logs has no panel-scoped keys yet, so its test checks
  focus only.

## 4. Testing

- [x] 4.1 Test closing a focused middle tab - focus moves to adjacent tab. Verify expected behavior.
  `util::shell::tab_focus::tests::closing_the_focused_middle_tab_focuses_the_tab_in_its_place`.
- [x] 4.2 Test closing the last tab - focus moves to sensible fallback. Verify focus is retained somewhere usable.
  `util::shell::tab_focus::tests::closing_the_focused_last_tab_focuses_its_neighbour` and `closing_a_groups_only_panel_focuses_another_group`.
- [x] 4.3 Test the reported case: close focused tab, then click another tab - it receives focus immediately. Verify the bug is fixed.
  `util::shell::tab_focus::tests::after_a_close_clicking_another_tab_focuses_it`.
- [x] 4.4 Run cargo test to ensure no regressions. Verify all tests pass. 766 passed, 2 ignored.
- [x] 4.5 Run linting/type checks as per project conventions. Verify clean build.