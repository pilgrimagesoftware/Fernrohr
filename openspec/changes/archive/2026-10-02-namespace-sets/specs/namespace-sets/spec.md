# Spec Delta

## Purpose

Let a user name a set of namespaces, keep it across launches, and switch a panel or a whole cluster
context to it, instead of retyping the same namespace list in every panel.

## ADDED Requirements

### Requirement: Named namespace sets

The user SHALL be able to create a named set of namespaces. Creating a set SHALL prefill the set
with the namespaces the focused namespaced panel is currently scoped to, so the common case is naming
what is already on screen. A set SHALL have a non-empty name that is unique among saved sets, and at
least one namespace; a set with no namespaces would be indistinguishable from "all namespaces" and
SHALL NOT be saved. The user SHALL be able to cancel creation, leaving no set behind. Sets SHALL
survive an application restart.

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

### Requirement: Removing a namespace set

The user SHALL be able to delete a saved set. Deleting a set SHALL remove it from the user's saved
sets and from the quick-selection picker. Deleting a set SHALL NOT change the namespace scope of any
open panel that was switched to it, and SHALL NOT change the context default; those keep the
namespaces they were given. When the user deletes a set, the editor SHALL ask for confirmation
first, and cancelling SHALL leave the set in place.

#### Scenario: Delete a set

- **WHEN** the user deletes `team-workloads` and confirms
- **THEN** `team-workloads` is no longer a saved set and is not offered for quick selection

#### Scenario: Deleting a set does not disturb switched panels

- **WHEN** a panel was switched to `team-workloads` containing `team-a`, and the user deletes
  `team-workloads`
- **THEN** the panel is still scoped to `team-a`
- **AND** the panel's namespace indicator reads as a count rather than a set name

#### Scenario: Deletion is confirmed

- **WHEN** the user deletes `team-workloads` and cancels the confirmation
- **THEN** `team-workloads` is still a saved set

### Requirement: Quick selection of a namespace set

A single command SHALL open a picker listing the user's sets in a stable order - the order they were
created in - with the digit `1` to `9` shown beside the sets in that order, a set's digit never
changing while the application runs. Sets past the ninth SHALL be listed without a digit. Typing a
digit SHALL switch to the set showing that digit; arrows and Enter SHALL also move and select;
Escape SHALL close the picker without changing any panel. The command SHALL be available wherever a
namespaced panel has focus, and SHALL appear in the command palette and be rebindable by id.

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
from the panel's scope or from the set. The user SHALL also be able to switch the same set onto every
namespaced panel in the window's active context and make it that context's default, so panels opened
later in that context start on the set. Switching SHALL NOT affect panels backed by a different
context, and cluster-scoped panels SHALL be unchanged by either action.

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

### Requirement: Every namespace set action is a named command

Each namespace-set action SHALL be registered in the command registry with a stable id, so it appears
in the command palette, accepts a `keymap.toml` override, and shows its current key hint in the
editor's title bar. A set's actions SHALL keep working when invoked from the palette rather than from
a key.

#### Scenario: Rebinding a namespace-set key

- **WHEN** the user assigns a different key to the quick-selection command's id in `keymap.toml` and
  restarts the application
- **THEN** the new key opens the picker, and the editor shows that key as its hint

#### Scenario: Invoking from the palette

- **WHEN** the user runs "Switch Namespace Set…" from the command palette instead of its key
- **THEN** the same picker opens
