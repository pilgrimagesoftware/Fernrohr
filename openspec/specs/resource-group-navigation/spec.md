# resource-group-navigation Specification

## Purpose
Sub-groups the Resource panel's Custom Resources section by Kubernetes API group, so a cluster
with many CRDs stays navigable by group rather than one long flat list.

## Requirements

### Requirement: Custom Resources sub-grouped by API group

Within the Resource panel's Custom Resources section, kinds SHALL be presented in subgroups
headed by their API group, ordered alphabetically by group name, with any core-group kind first
and kinds within a subgroup in the existing sort order. The top-level category sections SHALL be
unchanged, and no kind SHALL appear in more than one subgroup.

#### Scenario: Custom Resources shows API-group subgroups

- **WHEN** a cluster serves CRDs in two API groups
- **THEN** Custom Resources shows one subgroup per group, in alphabetical order by group name
- **AND** each kind appears under its own group's subgroup

### Requirement: Subgroup collapse and expand

Each subgroup SHALL have a header that collapses and expands its kinds. Every subgroup SHALL start
collapsed. Subgroup collapse state SHALL be held per window alongside section collapse state and
SHALL NOT be written to the user's preference file.

#### Scenario: Collapsing a subgroup hides its kinds

- **WHEN** the user collapses a subgroup
- **THEN** that subgroup's kinds are hidden and only its header remains visible
- **AND** other subgroups and sections are unaffected

#### Scenario: A new window starts collapsed

- **WHEN** a new window opens
- **THEN** every subgroup is collapsed, showing only its header

#### Scenario: A newly discovered group starts collapsed

- **WHEN** discovery reports a CRD in an API group not shown before
- **THEN** that group's subgroup appears collapsed

### Requirement: Filtering overrides collapsed subgroups

The Resource panel's filter SHALL match kinds across all subgroups regardless of collapsed
state, SHALL show only subgroups containing at least one match while a filter is active, and
SHALL temporarily expand any matching subgroup that was collapsed.

#### Scenario: Filter reveals a match inside a collapsed subgroup

- **WHEN** the user types a filter that matches a kind in a collapsed subgroup
- **THEN** that subgroup is shown expanded with the matching kind visible
- **AND** subgroups with no matching kind are hidden

#### Scenario: Clearing the filter restores prior collapsed state

- **WHEN** the user clears the filter after it temporarily expanded a collapsed subgroup
- **THEN** that subgroup returns to collapsed

### Requirement: Keyboard navigation across subgroups

Keyboard navigation in the Resource panel SHALL move through subgroup headers and visible kinds
in drawn order, and a keybinding SHALL toggle the collapsed state of the subgroup containing the
focused item.

#### Scenario: Arrow-key navigation crosses a subgroup boundary

- **WHEN** the user presses down-arrow while the last visible kind in a subgroup has focus
- **THEN** focus moves to the next subgroup's header

#### Scenario: Toggling the focused subgroup

- **WHEN** the user invokes "toggle group" while a kind within a subgroup has focus
- **THEN** that subgroup collapses or expands
- **AND** if it collapses, focus moves to its header

### Requirement: Collapse and expand all subgroups

The Resource panel SHALL provide "collapse all" and "expand all" commands, each bound to a key,
that set every Custom Resources subgroup's collapsed state at once. They SHALL NOT fire while the
filter has focus, and SHALL NOT change the top-level category sections.

#### Scenario: Expand all

- **WHEN** some subgroups are collapsed and the user invokes "expand all"
- **THEN** every subgroup is expanded

#### Scenario: Collapse all moves focus to a header

- **WHEN** a kind inside a subgroup has focus and the user invokes "collapse all"
- **THEN** every subgroup is collapsed
- **AND** focus moves to that kind's subgroup header
