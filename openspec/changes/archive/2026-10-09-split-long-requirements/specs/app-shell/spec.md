# Spec Delta

## MODIFIED Requirements

### Requirement: Window status bar

Every workspace window SHALL show a status bar along its bottom edge with one capsule per cluster
context the window uses, followed by the add-context control. Each capsule SHALL show, in order,
the context name, the name of its bound tunnel if any in square brackets, and an icon for its
connection state, and for any state other than connected, how long it has been in that state.

#### Scenario: One item per cluster in the window

- **WHEN** a window uses `cluster-a` (through `qa-bastion`) and `cluster-b` (direct)
- **THEN** its status bar shows a `cluster-a` capsule reading `cluster-a [qa-bastion]` followed by
  its state icon, a `cluster-b` capsule with no tunnel name, and no capsule for contexts only other
  windows use

#### Scenario: State shown as an icon

- **WHEN** the user hovers the state icon of a connected `cluster-b` capsule
- **THEN** a tooltip reads "Connected", and the capsule itself shows no state text

#### Scenario: Problem items first

- **WHEN** `cluster-b` is connected and `cluster-a` is paused
- **THEN** the `cluster-a` capsule is listed before the `cluster-b` capsule

#### Scenario: Closing panels keeps the item

- **WHEN** the user closes the last panel for a context the window still uses
- **THEN** that context's capsule stays in the window's status bar

#### Scenario: Each context appears once

- **WHEN** a window uses one context
- **THEN** that context's name and health appear in exactly one place in the window chrome

### Requirement: Keyboard focus navigation between panels

The application SHALL let the user move keyboard focus between the visible panels of a window
from the keyboard, as registered commands offered in the command palette, bound to default keys
the user's keymap can override, and listed in the application menu.

#### Scenario: Focus moves to the next panel

- **WHEN** a window shows two or more panels and one of them has keyboard focus
- **AND** the user invokes "focus next panel"
- **THEN** keyboard focus moves to the next panel in the order
- **AND** that panel's tab shows it has focus

#### Scenario: Focus wraps around

- **WHEN** the last panel in the order has keyboard focus
- **AND** the user invokes "focus next panel"
- **THEN** keyboard focus moves to the first panel
- **AND** invoking "focus previous panel" from the first panel moves focus to the last

#### Scenario: Previous reverses next

- **WHEN** the user invokes "focus next panel" and then "focus previous panel"
- **THEN** keyboard focus is back on the panel it started on

#### Scenario: No panel has focus yet

- **WHEN** no panel in the window has keyboard focus
- **AND** the user invokes "focus next panel"
- **THEN** keyboard focus moves to the first panel in the order

#### Scenario: Hidden panels are skipped

- **WHEN** a dock is collapsed, or another panel is zoomed
- **AND** the user moves focus with "focus next panel" or "focus previous panel"
- **THEN** focus only ever lands on panels that are visible

#### Scenario: Only one panel

- **WHEN** a window shows exactly one panel
- **AND** the user invokes "focus next panel" or "focus previous panel"
- **THEN** that panel has keyboard focus and nothing else changes

#### Scenario: Available from the command palette

- **WHEN** the user opens the command palette in a window with panels
- **THEN** "focus next panel" and "focus previous panel" are offered, each showing its key binding

### Requirement: Keyboard switching between the tabs of a group

The application SHALL let the user switch which tab a tab group displays from the keyboard: to
the next or previous tab, and to a tab by its position, as registered commands offered in the
command palette, bound to default keys the user's keymap can override. The tab shown by one of
these commands SHALL receive keyboard focus.

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

## ADDED Requirements

### Requirement: Status capsule secondary text is muted

In a status bar capsule, the tunnel name and the elapsed time SHALL render smaller than the
context name, in the data font and the theme's muted foreground color.

#### Scenario: Tunnel and elapsed time are secondary

- **WHEN** a capsule shows a bound tunnel's name and an elapsed time
- **THEN** both render smaller than the context name, in the data font and the theme's muted foreground color

### Requirement: Status capsule state icon has a tooltip

A status bar capsule's state icon SHALL have a tooltip naming the state and, for any state other
than connected, how long it has lasted.

#### Scenario: Hovering a connected capsule's icon

- **WHEN** the user hovers the state icon of a connected capsule
- **THEN** a tooltip reads "Connected"

### Requirement: Non-connected capsules are listed first

Status bar capsules in a non-connected state SHALL be listed before connected ones.

#### Scenario: A paused context comes first

- **WHEN** `cluster-b` is connected and `cluster-a` is paused
- **THEN** the `cluster-a` capsule is listed before the `cluster-b` capsule

### Requirement: Panel focus order is stable and wraps

Moving focus to the next or previous panel SHALL follow one stable order over the window's
visible panels, wrapping from the last panel to the first and from the first to the last.

#### Scenario: Next from the last panel

- **WHEN** the last panel in the order has keyboard focus and the user invokes "focus next panel"
- **THEN** keyboard focus moves to the first panel

### Requirement: Hidden panels never receive focus by panel navigation

Panels the user cannot see - in a collapsed dock, or behind another panel that is zoomed - SHALL
NOT receive focus from the next and previous panel commands.

#### Scenario: A collapsed dock is skipped

- **WHEN** a dock is collapsed and the user moves focus with "focus next panel"
- **THEN** focus only ever lands on panels that are visible

### Requirement: Tab switching acts on the focused tab group

The tab switching commands SHALL act on the focused tab group: the group whose displayed panel
keyboard focus is inside, or, when focus is outside every group, the first group in panel-focus
order.

#### Scenario: Focus outside the dock

- **WHEN** keyboard focus is in the Resource panel, the dock has panels, and the user invokes "next tab"
- **THEN** the first group in panel-focus order switches to its next tab
