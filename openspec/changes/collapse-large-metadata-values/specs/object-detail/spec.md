# Spec Delta

## ADDED Requirements

### Requirement: Large label and annotation values are shortened in the object viewer
The object panel's structured view SHALL shorten label and annotation chips exactly as the pod
detail panel does: a value longer than 20 characters or spanning more than one line shows its first
20 characters of the first line and an ellipsis, with the full value in a hover tooltip. A value the
panel redacts, such as a Secret's last-applied-configuration annotation, SHALL show only its
redacted placeholder in both the chip and the tooltip.

#### Scenario: An object's long annotation is shortened
- **WHEN** a Deployment's panel shows an annotation whose value is multi-line
- **THEN** its chip shows the key, the first 20 characters of the first line, and an ellipsis, and
  hovering it shows the full value

#### Scenario: A redacted annotation stays redacted
- **WHEN** a Secret's panel shows its last-applied-configuration annotation
- **THEN** neither the chip nor its tooltip contains any Secret value
