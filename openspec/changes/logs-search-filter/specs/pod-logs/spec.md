# Spec Delta

## ADDED Requirements

### Requirement: Log lines can be filtered by substring
The log panel SHALL let the user filter visible log lines to those containing a typed substring,
case-insensitive, without discarding the underlying unfiltered history.

#### Scenario: Filtering narrows visible lines
- **WHEN** the user types a substring into the log panel's filter
- **THEN** only lines containing that substring (case-insensitive) are visible

#### Scenario: Clearing the filter restores full history
- **WHEN** the user clears the filter
- **THEN** every received line is visible again, in original order
