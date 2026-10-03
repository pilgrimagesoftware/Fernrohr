# Spec Delta

## MODIFIED Requirements

### Requirement: Managed fields are collapsed by default

The Managed Fields tab SHALL show one disclosure row per field manager (manager, operation, time),
collapsed by default, expanding to that manager's fields, operable by keyboard as well as mouse.
The tab's content area SHALL NOT show a redundant "Managed Fields" section label beside the rows,
since the tab heading already identifies the content.

#### Scenario: Expanding one manager

- **WHEN** the user opens the Managed Fields tab and expands the `kubectl-client-side-apply` row
- **THEN** only that manager's fields are shown, and the other managers stay collapsed

#### Scenario: No duplicate label beside the content

- **WHEN** the user opens the Managed Fields tab
- **THEN** the content area shows only the disclosure rows, with no "Managed Fields" label to
  their left
