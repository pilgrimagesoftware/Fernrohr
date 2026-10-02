# Spec Delta

## ADDED Requirements

### Requirement: Spacing comes from one scale
Panel insets, card padding, table row height and the gaps between sections SHALL come from one
shared spacing scale rather than per-panel values, and SHALL scale with the text-size preference.
No panel's content SHALL sit closer than the scale's panel inset to the panel's edge, and no card's
content closer than its card padding to the card's edge.

#### Scenario: Content is inset from panel edges
- **WHEN** any panel is shown
- **THEN** its content is inset from the panel's edges by at least the panel inset

#### Scenario: Spacing follows text size
- **WHEN** the user increases the text size
- **THEN** insets, padding and row heights grow in proportion
