# Spec Delta

## MODIFIED Requirements

### Requirement: Named namespace sets

The user SHALL be able to create a named set of namespaces. Creating a set SHALL prefill the set
with the namespaces the focused namespaced panel is currently scoped to, so the common case is
naming what is already on screen.

#### Scenario: Name the current scope

- **WHEN** a Pods panel is scoped to `team-a` and `team-b`, and the user creates a set named
  `team-workloads`
- **THEN** a set `team-workloads` containing `team-a` and `team-b` is saved
- **AND** the panel is still scoped to both namespaces

#### Scenario: Duplicate names are rejected

- **WHEN** a set named `team-workloads` exists and the user creates another set with that name
- **THEN** the second set is not saved and the editor says the name is taken

#### Scenario: An empty set cannot be saved

- **WHEN** the user creates a set with no namespaces in it
- **THEN** the set is not saved and the editor says a set needs at least one namespace

#### Scenario: Sets survive a restart

- **WHEN** the user saves a set named `team-workloads`, then restarts the application
- **THEN** `team-workloads` is listed among the user's sets

#### Scenario: Cancelling creates nothing

- **WHEN** the user starts creating a set and dismisses the editor without saving
- **THEN** no new set exists

### Requirement: Quick selection of a namespace set

A single command SHALL open a picker listing the user's sets in a stable order - the order they were
created in - with the digit `1` to `9` shown beside the sets in that order, a set's digit never
changing while the application runs. Sets past the ninth SHALL be listed without a digit.

#### Scenario: Two keystrokes to switch

- **WHEN** the sets are `team-workloads` first and `payments` second, and the user runs the quick
  selection command and presses `2`
- **THEN** the focused panel is switched to `payments`

#### Scenario: The tenth set has no digit

- **WHEN** the user has ten sets and opens the picker
- **THEN** every set is listed and the tenth carries no digit
- **AND** that set is still selectable by highlighting it and pressing Enter

#### Scenario: Escape changes nothing

- **WHEN** the picker is open and the user presses Escape
- **THEN** the picker closes and the focused panel's scope is unchanged

#### Scenario: Digits are shown only where the command works

- **WHEN** the picker is opened from a panel with no namespace scope, such as a cluster-scoped panel
- **THEN** no digits are shown and selecting a set reports that it needs a namespaced panel

### Requirement: Switching a panel to a namespace set

Applying a set SHALL set the focused namespaced panel's scope to an include set of exactly that
set's namespaces, so the panel shows resources in those namespaces and no others. A namespace in the
set that the connected cluster does not have SHALL simply match nothing, and SHALL NOT be dropped
from the panel's scope or from the set.

#### Scenario: Switch the focused panel

- **WHEN** the user applies `team-workloads` containing `team-a` and `team-b` to a Pods panel
- **THEN** the panel shows Pods from `team-a` and `team-b` only
- **AND** the panel persists with that scope

#### Scenario: A namespace absent from the cluster stays in scope

- **WHEN** `team-workloads` contains `team-a` and `retired-team`, `retired-team` does not exist in the
  connected cluster, and the user applies the set
- **THEN** the panel is still scoped to both `team-a` and `retired-team`
- **AND** the saved set still contains `retired-team`

#### Scenario: Switch every panel in the context

- **WHEN** two Pods panels are open against the same cluster context and the user applies
  `team-workloads` to the context
- **THEN** both panels show Pods from `team-a` and `team-b`
- **AND** a Pods panel opened later in that context starts scoped to the set

#### Scenario: Switching does not cross contexts

- **WHEN** panels are open against two different cluster contexts and the user applies
  `team-workloads` to the context in one
- **THEN** the panels in the other context are unchanged

#### Scenario: Cluster-scoped panels are untouched

- **WHEN** a cluster-scoped Nodes panel is open and the user applies `team-workloads` to the context
- **THEN** the Nodes panel is unchanged

## ADDED Requirements

### Requirement: A namespace set needs a unique name and a namespace

A namespace set SHALL have a non-empty name that is unique among saved sets, and at least one
namespace; a set with no namespaces would be indistinguishable from "all namespaces" and SHALL NOT
be saved.

#### Scenario: A taken name is refused

- **WHEN** a set named `team-workloads` exists and the user saves another set with that name
- **THEN** the second set is not saved and the editor says the name is taken

### Requirement: Cancelling set creation leaves nothing

The user SHALL be able to cancel creating a namespace set, leaving no set behind.

#### Scenario: Dismissing the editor

- **WHEN** the user starts creating a set and dismisses the editor without saving
- **THEN** no new set exists

### Requirement: Namespace sets survive a restart

Namespace sets SHALL survive an application restart.

#### Scenario: A saved set after a restart

- **WHEN** the user saves a set and then restarts the application
- **THEN** the set is listed among the user's sets

### Requirement: Selecting from the namespace set picker

In the namespace set picker, typing a digit SHALL switch to the set showing that digit; arrows and
Enter SHALL also move and select; Escape SHALL close the picker without changing any panel.

#### Scenario: Arrows and Enter select a set

- **WHEN** the picker is open and the user moves the highlight to a set with the arrows and presses
  Enter
- **THEN** the focused panel is switched to that set

### Requirement: The namespace set picker command is available to namespaced panels

The namespace set quick selection command SHALL be available wherever a namespaced panel has focus,
and SHALL appear in the command palette and be rebindable by id.

#### Scenario: Listed in the palette

- **WHEN** a namespaced panel has focus and the user opens the command palette
- **THEN** the quick selection command is listed, and its key can be rebound by the command's id

### Requirement: Switching a context to a namespace set

The user SHALL also be able to switch a namespace set onto every namespaced panel in the window's
active context and make it that context's default, so panels opened later in that context start on
the set. Switching SHALL NOT affect panels backed by a different context, and cluster-scoped panels
SHALL be unchanged by applying a set to the focused panel or to the context.

#### Scenario: A context's panels and its later panels

- **WHEN** two Pods panels are open against one context and the user switches `team-workloads` onto
  that context
- **THEN** both panels show Pods from the set's namespaces, and a Pods panel opened later in that
  context starts scoped to the set
