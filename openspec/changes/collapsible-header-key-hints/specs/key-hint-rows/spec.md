# Spec Delta

## Purpose

How a panel's row of key hints behaves when space is tight: collapsing to its essential hints
instead of growing taller, with every hidden hint still reachable by mouse and keyboard.

## ADDED Requirements

### Requirement: A hint row collapses instead of wrapping past three lines
A panel's key-hint row SHALL collapse when its hints would wrap onto more than three lines at the
row's current width. A collapsed row SHALL show only its essential hints followed by an overflow
control. It SHALL expand again only once all its hints fit in three lines at the current width.

#### Scenario: A narrow detail panel
- **WHEN** a pod detail panel is narrow enough that its header's hints would wrap onto four lines
- **THEN** the header shows only its essential hints and the overflow control, and stays no
  taller than three lines of hints

#### Scenario: Widening the panel
- **WHEN** a collapsed row's panel widens until every hint fits in three lines
- **THEN** the row shows every hint again and no overflow control

#### Scenario: Room to spare
- **WHEN** a row's hints fit in three lines or fewer
- **THEN** every hint shows and there is no overflow control

### Requirement: Each row marks its essential hints
Each hint row SHALL mark which of its hints are essential, so a collapsed row keeps the hints that
row's panel most needs. A row that marks none essential SHALL keep its first hint. Essential hints
SHALL keep their order in the row.

#### Scenario: A collapsed detail header
- **WHEN** an object detail panel's header is collapsed
- **THEN** it still shows its view toggle hint

#### Scenario: A collapsed list
- **WHEN** a Pods panel's hint row is collapsed
- **THEN** it still shows its Filter and Describe hints

### Requirement: The overflow control reveals the hidden hints by mouse and keyboard
A collapsed row's overflow control SHALL be an icon button whose tooltip lists the hidden hints,
each with its live key from the keymap. It SHALL be a tab stop. Enter or Space on it SHALL open
the Key Hints overlay, which lists every command available where focus is. Every hidden hint's
key SHALL still work while hidden.

#### Scenario: Hovering the control
- **WHEN** the user hovers a collapsed row's overflow control
- **THEN** a tooltip lists each hidden hint with its current key

#### Scenario: Reaching it by keyboard
- **WHEN** the user tabs to a collapsed row's overflow control and presses Enter
- **THEN** the Key Hints overlay opens, listing the hidden hints' commands with their keys

#### Scenario: A hidden hint's key
- **WHEN** a row is collapsed and the user presses the key of a hint it hides
- **THEN** that hint's command runs as it would with the row expanded

### Requirement: Which hint rows collapse
The collapse SHALL apply to the pod detail and object detail headers' hints and to the Pods,
object list and events browser panels' hint rows. The shell panel's hint row SHALL NOT collapse,
since it always lists its reserved keys. Keys shown beside individual controls, as in dialogs,
the Logs panel's control bar and the cluster picker, are not hint rows and SHALL NOT collapse.

#### Scenario: The shell panel keeps its keys
- **WHEN** a shell panel is narrow enough that its hint row would wrap past three lines
- **THEN** its hint row still shows every reserved key

### Requirement: A hidden hint still counts as listed
A hint hidden by the collapse SHALL still count as appearing in its panel's hint row. It is listed
by the row's overflow control, and its command remains in the command palette and the Key Hints
overlay.

#### Scenario: A command a spec requires in the hint row
- **WHEN** a Pods panel's row is collapsed and its Port-forward hint is hidden
- **THEN** Port-forward is listed in the overflow control's tooltip and the Key Hints overlay, and
  its key still runs it
