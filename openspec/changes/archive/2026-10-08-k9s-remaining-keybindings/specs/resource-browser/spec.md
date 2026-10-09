# Spec Delta

## ADDED Requirements

### Requirement: Numeric namespace quick-jump

A resource-browser panel SHALL provide numeric commands that select a namespace by its position in
the panel's current namespace list, so a frequently used namespace can be reached in one keystroke.

#### Scenario: Jump to a namespace by position

- **WHEN** the user invokes quick-jump `3` and the panel's namespace list has at least three entries
- **THEN** the panel switches to the namespace at position three in that list

#### Scenario: Position beyond the list

- **WHEN** the user invokes a quick-jump position beyond the number of namespaces in the list
- **THEN** the command has no effect and the panel's namespace selection is unchanged
