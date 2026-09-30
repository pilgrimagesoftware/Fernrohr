# Spec Delta

## ADDED Requirements

### Requirement: Keyboard focus navigation between panels

The application SHALL let the user move keyboard focus between the visible panels of a window
from the keyboard, as registered commands offered in the command palette, bound to default keys
the user's keymap can override, and listed in the application menu. "Next" and "previous" SHALL
follow one stable order over the window's visible panels, wrapping from the last panel to the
first and from the first to the last. Panels the user cannot see - in a collapsed dock, or behind
another panel that is zoomed - SHALL NOT receive focus this way.

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

### Requirement: An opened panel takes focus

When the user opens a panel, or shows one that is already open, the application SHALL move
keyboard focus to that panel, whichever route opened it (a click, a command, or a panel's own
shortcut).

#### Scenario: Opening a panel from the keyboard focuses it

- **WHEN** the user opens a pod's detail panel from a Pods panel with its keyboard shortcut
- **THEN** the pod detail panel has keyboard focus
- **AND** its own shortcuts respond without the user clicking it first

#### Scenario: Re-showing an open panel focuses it

- **WHEN** the user asks for a panel that is already open in the window
- **THEN** that panel is brought to the front of its tab group and has keyboard focus
