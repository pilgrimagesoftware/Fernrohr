# Spec Delta

## MODIFIED Requirements

### Requirement: Namespace scoping

A resource-browser panel SHALL let the user view a single namespace, all namespaces, or an
include/exclude set of namespaces, and SHALL persist the selection with the panel.

An include/exclude set is one mode (include or exclude) plus a set of namespace names. "Include"
shows only resources in those namespaces; "exclude" shows resources from every namespace except
those. Adding a namespace to the current set SHALL NOT discard the rest of the set.

#### Scenario: Restrict to one namespace

- **WHEN** the user selects namespace `kube-system`
- **THEN** the table shows only Pods in `kube-system`

#### Scenario: All namespaces

- **WHEN** the user selects "all namespaces"
- **THEN** the table shows Pods from every namespace the user can list

#### Scenario: Include a namespace into the set

- **WHEN** the panel's scope is an include set containing `team-a`, and the user includes `team-b`
- **THEN** the table shows Pods from both `team-a` and `team-b`, and the panel's scope persists as
  an include set containing both

#### Scenario: Exclude a namespace from the set

- **WHEN** the panel's scope is "all namespaces" and the user excludes `kube-system`
- **THEN** the table shows Pods from every namespace except `kube-system`, and the panel's scope
  persists as an exclude set containing `kube-system`

#### Scenario: Removing the last excluded namespace reverts to all

- **WHEN** the panel's scope is an exclude set containing only `kube-system`, and the user
  re-includes `kube-system`
- **THEN** the panel's scope reverts to "all namespaces"

## ADDED Requirements

### Requirement: Namespace scoping propagates across panels for a context

Including or excluding a namespace "for all" SHALL apply the same change to every open panel
backed by the same cluster context, and SHALL update that context's default namespace scope so
panels opened afterward in that context start with the updated include/exclude set.

#### Scenario: Include for all updates every open panel

- **WHEN** two Pods panels are open against the same cluster context, and the user includes
  `team-a` "for all" from one of them
- **THEN** both panels' scopes become an include set containing `team-a`

#### Scenario: New panel in the context inherits the propagated scope

- **WHEN** the user excludes `kube-system` "for all" in a context, then opens a new Pods panel
  against that same context
- **THEN** the new panel opens already scoped to exclude `kube-system`

#### Scenario: Propagation does not cross contexts

- **WHEN** panels are open against two different cluster contexts, and the user includes
  `team-a` "for all" in the first context
- **THEN** panels in the second context are unaffected
