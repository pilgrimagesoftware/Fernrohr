# Spec Delta

## ADDED Requirements

### Requirement: Loading state
While a list panel has not finished its first list, its table area SHALL show a loading indicator naming the kind being loaded instead of an empty table, and SHALL show how many rows have arrived once any have. The indicator SHALL appear only if loading lasts longer than a short delay.

#### Scenario: Slow first load
- **WHEN** a Pods panel opens and its first list takes several seconds
- **THEN** the table area shows "Loading Pods…" with a spinner, and then a count of Pods received so far, until the list completes

#### Scenario: Fast load never flickers
- **WHEN** a panel's first list completes within the delay
- **THEN** no loading indicator is ever drawn

### Requirement: Empty state
When a list panel's list has completed with no rows, the table area SHALL say that there are none, naming the kind and its namespace scope, so it is never mistaken for loading. When a filter hides every row, it SHALL say that no rows match the filter.

#### Scenario: Empty namespace
- **WHEN** a Pods panel scoped to `team-a` finishes loading with no Pods
- **THEN** the table area reads "No Pods in team-a"

#### Scenario: Filter hides everything
- **WHEN** the panel has rows but the filter text matches none of them
- **THEN** the table area says no rows match the filter

### Requirement: Refreshing keeps rows visible
When a list panel relists while it already shows rows, for example after a watch restart or when a paused connection resumes, it SHALL keep showing its rows and SHALL show a small refreshing indicator in the panel header until the relist completes.

#### Scenario: Watch restart
- **WHEN** a loaded Pods panel's watch restarts and relists
- **THEN** the current rows stay visible with a refreshing indicator in the header until the relist completes

#### Scenario: Scope change is instant
- **WHEN** the user changes a loaded Pods panel from `team-a` to all namespaces
- **THEN** the rows update at once without a refreshing indicator, since the panel already holds every namespace's rows
