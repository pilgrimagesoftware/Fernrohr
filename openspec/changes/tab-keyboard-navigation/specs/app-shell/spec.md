# Spec Delta

## ADDED Requirements

### Requirement: Keyboard switching between the tabs of a group

The application SHALL let the user switch which tab a tab group displays from the keyboard: to
the next or previous tab, and to a tab by its position. These SHALL be registered commands
offered in the command palette, bound to default keys the user's keymap can override. They act
on the focused tab group: the group whose displayed panel keyboard focus is inside, or, when
focus is outside every group, the first group in panel-focus order. The tab shown by one of these
commands SHALL receive keyboard focus.

#### Scenario: Next tab shows and focuses the following tab

- **WHEN** a tab group has two or more tabs and its displayed panel has keyboard focus
- **AND** the user invokes "next tab"
- **THEN** the group displays the tab after the previous one
- **AND** that tab's panel has keyboard focus

#### Scenario: Next and previous wrap within the group

- **WHEN** the group's last tab is displayed and the user invokes "next tab"
- **THEN** the group displays its first tab
- **AND** invoking "previous tab" from the first tab displays the last

#### Scenario: A hidden tab becomes reachable

- **WHEN** a pod detail panel is a tab behind the Pods panel in the same group
- **AND** the user invokes "next tab" or "previous tab" until it is displayed
- **THEN** the pod detail panel is displayed and has keyboard focus, without the user clicking

#### Scenario: Select a tab by position

- **WHEN** the focused group has at least three tabs
- **AND** the user invokes "select tab 3"
- **THEN** the group displays its third tab, and that tab has keyboard focus
- **AND** invoking "select tab 9" displays the group's last tab, however many tabs it has

#### Scenario: A position past the last tab does nothing

- **WHEN** the focused group has two tabs
- **AND** the user invokes "select tab 5"
- **THEN** the displayed tab and keyboard focus are unchanged

#### Scenario: A group with one tab

- **WHEN** the focused group has exactly one tab
- **AND** the user invokes "next tab" or "previous tab"
- **THEN** that tab stays displayed and has keyboard focus

#### Scenario: Focus outside the dock

- **WHEN** keyboard focus is in the Resource panel and the dock has panels
- **AND** the user invokes "next tab"
- **THEN** the first group in panel-focus order switches to its next tab, and that tab has keyboard
  focus

#### Scenario: Offered in the command palette

- **WHEN** the user opens the command palette in a window with panels
- **THEN** "next tab", "previous tab" and "select tab 1" through "select tab 9" are offered, each
  showing its key binding
