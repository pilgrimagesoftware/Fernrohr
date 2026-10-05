# Spec Delta

## ADDED Requirements

### Requirement: Opening a list row in the background

Every list panel SHALL offer an "Open in Background" command that opens the selected row's
detail panel the way activating the row does, except that the new panel is added as an inactive
tab and keyboard focus and the list's selection stay where they were. The command SHALL be bound
by default to the platform modifier with Enter (`cmd-enter` on macOS, `ctrl-enter` elsewhere),
SHALL NOT fire while a text field has focus, and SHALL appear in the command palette and the
panel's hint row. A click on a row with the platform modifier held, or a middle-click on a row,
SHALL do the same for the clicked row without changing the selection. If the object's panel is
already open, the command SHALL change nothing: it neither activates that tab nor moves focus.

#### Scenario: Background open from the keyboard

- **WHEN** a Pods panel has focus with a row selected and the user presses `cmd-enter`
- **THEN** that pod's detail panel opens as an inactive tab, and the Pods panel keeps keyboard focus and the same selected row

#### Scenario: Background open with the mouse

- **WHEN** the user `cmd`-clicks the third row of a Deployments list whose first row is selected
- **THEN** the third row's Deployment opens as an inactive tab, and the first row stays selected with focus in the list

#### Scenario: Middle-click

- **WHEN** the user middle-clicks a row in any list panel
- **THEN** that row's object opens in the background, as with the modifier-click

#### Scenario: The list stays visible in its own group

- **WHEN** a background open places the new panel in the same tab group as the list
- **THEN** the list remains the active tab of that group

#### Scenario: Already open

- **WHEN** the user opens in the background an object whose panel is already open
- **THEN** no panel opens, no tab is activated, and focus stays in the list

#### Scenario: Typing is not intercepted

- **WHEN** a list panel's filter field has focus and the user presses `cmd-enter`
- **THEN** no panel opens
