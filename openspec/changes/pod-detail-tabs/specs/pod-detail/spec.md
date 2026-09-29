# Spec Delta

## ADDED Requirements

### Requirement: Structured fields are organized into tabs
The pod detail panel's structured field view SHALL organize its fields into tabs rather than one
continuous scroll, and SHALL let the user switch tabs by keystroke as well as by clicking.

#### Scenario: Switching tabs shows only that tab's fields
- **WHEN** the user switches to a tab
- **THEN** only that tab's fields are visible, and the panel does not require scrolling past
  unrelated fields to reach them

#### Scenario: Every field still appears somewhere
- **WHEN** every tab's fields are considered together
- **THEN** every field `pod_fields` projects for the pod appears in exactly one tab
