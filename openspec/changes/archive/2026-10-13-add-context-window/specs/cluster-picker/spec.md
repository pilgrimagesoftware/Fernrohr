## ADDED Requirements

### Requirement: Long context names stay usable

The picker's context list SHALL grow with its window so long context names have room. A name too
long for the row SHALL be shortened with an ellipsis and shown in full on hover, and each row's
tunnel selector SHALL stay fully visible and usable at every window width the picker allows.

#### Scenario: A long name keeps its tunnel selector

- **WHEN** the picker lists a context whose name is longer than the row
- **THEN** the name ends in an ellipsis, hovering it shows the full name, and the row's tunnel
  selector is fully inside the picker

#### Scenario: A wider window shows more of the name

- **WHEN** the user widens the picker's window
- **THEN** the context list widens with it and shows more of each long name
