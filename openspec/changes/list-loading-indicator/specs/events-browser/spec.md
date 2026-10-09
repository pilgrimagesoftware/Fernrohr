# Spec Delta

## ADDED Requirements

### Requirement: Events browser loading and empty states
The events browser SHALL show the same delayed loading indicator, empty state, and header refreshing indicator as resource list panels, naming events as the kind being loaded.

#### Scenario: Loading events
- **WHEN** an events browser opens and its first list takes several seconds
- **THEN** it shows "Loading events…" with a spinner, and then a count of events received so far, until the list completes
