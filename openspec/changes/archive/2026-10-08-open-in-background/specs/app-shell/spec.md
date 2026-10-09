# Spec Delta

## MODIFIED Requirements

### Requirement: An opened panel takes focus

When the user opens a panel, or shows one that is already open, the application SHALL move
keyboard focus to that panel, whichever route opened it (a click, a command, or a panel's own
shortcut). The one exception is a background open (a modifier-click, a middle-click, or the "Open
in Background" command): it SHALL add the panel as an inactive tab, or leave an already-open panel
as it is, and SHALL NOT move keyboard focus.

#### Scenario: Opening a panel from the keyboard focuses it

- **WHEN** the user opens a pod's detail panel from a Pods panel with its keyboard shortcut
- **THEN** the pod detail panel has keyboard focus
- **AND** its own shortcuts respond without the user clicking it first

#### Scenario: Re-showing an open panel focuses it

- **WHEN** the user asks for a panel that is already open in the window
- **THEN** that panel is brought to the front of its tab group and has keyboard focus

#### Scenario: A background open leaves focus alone

- **WHEN** the user opens a pod's detail panel in the background from a Pods panel
- **THEN** the pod detail panel exists as an inactive tab, and the Pods panel still has keyboard focus
