# Spec Delta

## MODIFIED Requirements

### Requirement: Editing a set's namespaces

The user SHALL be able to add and remove namespaces in a saved set. The editor SHALL list the
namespaces the connected cluster reports, filterable by typing as in the namespace picker, and
SHALL mark each namespace that is currently in the set. Adding or removing a namespace SHALL mark
the set changed and saving SHALL persist it immediately, so no further confirm step is needed.
Editing a set SHALL NOT change the scope of any panel already switched to it.

#### Scenario: Add a namespace to a set

- **WHEN** the user edits `team-workloads`, types `team` in the filter, and adds `team-c`
- **THEN** `team-c` is marked as being in the set
- **AND** saving makes `team-workloads` contain `team-a`, `team-b` and `team-c`

#### Scenario: Remove a namespace from a set

- **WHEN** the user edits `team-workloads` and removes `team-b`
- **THEN** saving leaves `team-workloads` containing only `team-a`

#### Scenario: Editing a set leaves switched panels alone

- **WHEN** a panel is switched to `team-workloads` containing `team-a`, and the user then adds
  `team-c` to that set
- **THEN** the panel is still scoped to `team-a` alone

#### Scenario: A set's namespaces may come from another cluster

- **WHEN** the connected cluster has no `team-c`, and the user edits a set to contain `team-c`
- **THEN** the editor lists `team-c` as being in the set, marked as absent from this cluster, and
  saving still succeeds

#### Scenario: Shift-click selects a range

- **WHEN** the user clicks `team-a` in the editor's namespace list, then shift-clicks `team-f`
  three rows below it
- **THEN** every namespace between `team-a` and `team-f`, inclusive, is added to the set
- **AND** a namespace already in the set within that range stays in the set

#### Scenario: Shift-click ranges apply to removal too

- **WHEN** every namespace from `team-a` to `team-f` is in the set, the user clicks `team-a`, then
  shift-clicks `team-f`
- **THEN** every namespace between `team-a` and `team-f`, inclusive, is removed from the set

#### Scenario: Shift-click with no prior click selects from the top

- **WHEN** the user shift-clicks a row with no earlier click in the list this time the editor was
  opened
- **THEN** every namespace from the first listed row to the shift-clicked row, inclusive, is added
  to the set

#### Scenario: A mixed range follows the anchor row's action

- **WHEN** `team-a` is in the set and `team-b` through `team-f` are not, and the user clicks
  `team-a` (removing it), then shift-clicks `team-f`
- **THEN** every namespace from `team-a` to `team-f` is removed from the set, matching the click
  on `team-a` rather than each row's own prior state
