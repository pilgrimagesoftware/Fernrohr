# Spec Delta

## ADDED Requirements

### Requirement: The namespace scope indicator names a matching set

A namespaced panel's namespace scope indicator SHALL show a saved namespace set's name when the
panel's scope is an include set whose namespaces are exactly that set's namespaces. Otherwise the
indicator SHALL read as before, as a count of included namespaces or as "all namespaces". Because a
panel stores the namespaces it was switched to rather than a reference to the set, deleting or
editing a set SHALL leave an already-switched panel's scope and indicator unchanged until the panel is
switched to a set again.

#### Scenario: A switched panel names its set

- **WHEN** the set `team-workloads` contains `team-a` and `team-b`, and a panel is switched to it
- **THEN** the panel's namespace indicator reads `team-workloads`

#### Scenario: A hand-built scope reads as a count

- **WHEN** the user builds an include set of `team-a` and `team-b` by hand and no saved set has
  exactly those namespaces
- **THEN** the panel's namespace indicator reads as a count, not a set name

#### Scenario: An edited set no longer matches

- **WHEN** a panel is switched to `team-workloads` containing `team-a`, and the user then adds
  `team-c` to that set
- **THEN** the panel's namespace indicator reads as a count, while the set itself shows three
  namespaces
