# Spec Delta

## Purpose

Organizes the resource kind picker's discovered kinds into sections by Kubernetes API group, so
that a cluster with many CRDs stays navigable by group rather than one long flat list.

## ADDED Requirements

### Requirement: Kinds grouped by API group

The resource kind picker SHALL present discovered kinds in sections headed by their API group,
with the core group shown first and all other groups ordered alphabetically after it, and kinds
within a group shown in the picker's existing sort order.

#### Scenario: Picker opens with grouped sections

- **WHEN** the user opens the resource kind picker for a cluster with core kinds and two CRD
  groups
- **THEN** the picker shows a core section first, followed by the two CRD groups' sections in
  alphabetical order by group name
- **AND** each kind appears under its own group's section, not under core

### Requirement: Group collapse and expand

The resource kind picker SHALL let the user collapse or expand any group's section, and SHALL
remember each group's collapsed state for the current cluster connection for the rest of the
session.

#### Scenario: Collapsing a group hides its kinds

- **WHEN** the user collapses a group's section
- **THEN** that group's kinds are hidden and only its header remains visible
- **AND** other groups' sections are unaffected

#### Scenario: Collapsed state persists across picker reopen in the same session

- **WHEN** the user collapses a group, closes the picker, and reopens it for the same cluster
  connection
- **THEN** the previously collapsed group is still collapsed

### Requirement: Filtering overrides collapsed groups

The resource kind picker's text filter SHALL match kinds across all groups regardless of their
collapsed state, SHALL show only groups containing at least one match while a filter is active,
and SHALL temporarily expand any matching group that was collapsed.

#### Scenario: Filter reveals a match inside a collapsed group

- **WHEN** the user types a filter that matches a kind in a collapsed group
- **THEN** that group's section is shown expanded with the matching kind visible
- **AND** groups with no matching kind are hidden entirely

#### Scenario: Clearing the filter restores prior collapsed state

- **WHEN** the user clears the filter after it temporarily expanded a collapsed group
- **THEN** that group returns to collapsed

### Requirement: Keyboard navigation across groups

The resource kind picker SHALL support keyboard navigation (arrow keys and type-ahead) that moves
between visible kinds in the order their groups and sections are drawn, and SHALL provide a
keybinding that toggles the collapsed state of the group containing the currently focused item.

#### Scenario: Arrow-key navigation crosses a group boundary

- **WHEN** the user presses the down-arrow key while the last visible kind in a group's section
  has focus
- **THEN** focus moves to the first visible kind in the next section

#### Scenario: Toggling the focused group's collapsed state

- **WHEN** the user invokes the "toggle group" action while a kind within a group has focus
- **THEN** that group collapses or expands
- **AND** if it collapses, focus moves to that group's header
