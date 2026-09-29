# Spec Delta

## Purpose

A dockable panel showing one Pod's description or YAML, independent of the Pods list panel it
was opened from, so detail can be viewed, moved, and closed like any other panel.

## ADDED Requirements

### Requirement: Detail opens as its own panel
Requesting a Pod's description or YAML SHALL open a dockable panel scoped to that specific pod,
distinct from the Pods list panel it was requested from.

#### Scenario: Describing a pod opens a detail panel
- **WHEN** the user requests a pod's description from a Pods list
- **THEN** a new panel opens showing that pod's description, and the Pods list panel remains
  showing only its table

#### Scenario: Requesting YAML opens a detail panel
- **WHEN** the user requests a pod's YAML from a Pods list
- **THEN** a new panel opens showing that pod's YAML

### Requirement: Reselecting the same pod focuses its existing panel
Requesting detail for a pod that already has an open detail panel for the same cluster SHALL
focus that panel rather than opening a duplicate.

#### Scenario: Same pod requested twice
- **WHEN** a pod's detail panel is already open and the user requests that same pod's detail
  again
- **THEN** the existing panel is focused and no second panel opens

#### Scenario: Different pods each get their own panel
- **WHEN** the user requests detail for two different pods in the same cluster
- **THEN** two distinct detail panels are open, each scoped to its own pod

### Requirement: Detail panel survives its source pod's disappearance
If the pod a detail panel is scoped to no longer exists in the cluster, the panel SHALL show
that the pod no longer exists rather than erroring or closing unexpectedly.

#### Scenario: Pod deleted while its detail panel is open
- **WHEN** a pod is deleted from the cluster while its detail panel is open
- **THEN** the panel shows that the pod no longer exists, and remains open until the user closes
  it
