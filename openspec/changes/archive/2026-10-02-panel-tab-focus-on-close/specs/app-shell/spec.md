# Spec Delta

## ADDED Requirements

### Requirement: The active panel tab holds keyboard focus
Whenever a docked panel tab becomes active - by clicking its tab, clicking inside its content,
keyboard tab navigation, opening a new panel, restoring a layout, or closing the tab that was
focused - that panel SHALL receive keyboard focus at once, so its keys work without a further
click or tab switch. Keyboard focus SHALL NOT be left on no panel while any panel is open.

#### Scenario: A tab accepts focus the first time it is clicked
- **WHEN** the user clicks a panel tab that has not had focus since the window opened
- **THEN** that panel has keyboard focus and its keys work immediately, without first switching
  to another tab and back

#### Scenario: A newly opened panel is focused
- **WHEN** the user opens a panel from the Resource panel or the command palette
- **THEN** the new panel's tab is active and the panel has keyboard focus

#### Scenario: Closing the focused tab transfers focus
- **WHEN** the user closes the panel tab that has keyboard focus and other panels remain open
- **THEN** focus moves to the tab that becomes active in its place, or to another open panel if
  its tab group is now empty, and is never lost

#### Scenario: Clicking a tab after a close gives it focus
- **WHEN** a tab has just been closed and the user clicks another panel tab
- **THEN** that tab's panel receives keyboard focus and keyboard input works immediately
