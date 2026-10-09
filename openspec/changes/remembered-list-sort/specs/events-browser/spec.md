# Spec Delta

## ADDED Requirements

### Requirement: Remembered events sort

The application SHALL remember the last sort the user chose in an events browser, across contexts, windows, and restarts. A newly opened events browser with no sort of its own SHALL start with the remembered sort, or with age ascending (newest first) when none is remembered. Returning a column to the default sort in an events browser SHALL restore age ascending.

#### Scenario: New events browser uses the remembered sort

- **WHEN** the user sorts an events browser by Reason and later opens an events browser for another context
- **THEN** the new events browser opens sorted by Reason

#### Scenario: Default unchanged

- **WHEN** the user has never sorted an events browser and opens one
- **THEN** it opens sorted by age ascending (newest first)
