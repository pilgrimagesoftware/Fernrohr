# Spec Delta

## ADDED Requirements

### Requirement: Large label and annotation values are shortened
In the pod detail panel's structured view, a label or annotation whose value is longer than 20
characters or spans more than one line SHALL render as a chip showing its key, `=`, the first 20
characters of the value's first line, and an ellipsis. Hovering that chip SHALL show the full value
in a tooltip. Chips SHALL NOT expand in place, and the YAML view SHALL keep showing every value in
full.

#### Scenario: A multi-line annotation is shortened
- **WHEN** a pod has an annotation whose value is multi-line JSON
- **THEN** its chip is one line tall, showing the key, `=`, the first 20 characters of the value's
  first line, and an ellipsis

#### Scenario: A long single-line annotation is shortened
- **WHEN** a pod has an annotation whose value is one line of 300 characters
- **THEN** its chip shows only the first 20 characters of the value and an ellipsis

#### Scenario: The full value is in a tooltip
- **WHEN** the user hovers a shortened chip
- **THEN** a tooltip shows the annotation's full value

#### Scenario: Short values are unchanged
- **WHEN** a label's value is 20 characters or fewer on one line
- **THEN** its chip shows the full `key=value` and has no tooltip
