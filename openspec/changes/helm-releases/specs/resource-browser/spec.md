# Spec Delta

## MODIFIED Requirements

### Requirement: Resource kinds are grouped into collapsible sections

The Resource panel SHALL present every discovered kind under exactly one named
section, rather than as one flat list, and SHALL show no kind twice. No kind may
be dropped by grouping: the union of the sections' kind rows is exactly the set of
kinds the cluster's API discovery reported.

A kind's section SHALL be determined by its API group **and** its resource
plural together. A section SHALL NOT be inferred from the API group alone.

The Workloads section SHALL also hold one row that is not a discovered kind, Helm
Releases (`helm-releases`), last in the section. It SHALL be navigable, filterable
and openable like a kind row, and SHALL NOT count as a kind in the section's count.

#### Scenario: Grouping loses no kind

- **WHEN** a cluster's discovery reports a kind that matches no known section
- **THEN** that kind appears under Custom Resources
- **AND** the total count of kind rows across all sections equals the number of
  discovered kinds

#### Scenario: Opening a kind still works from a section

- **WHEN** the user opens a kind from within a section
- **THEN** the same panel opens as it would from a flat list

#### Scenario: Helm Releases from the Resource panel

- **WHEN** the user filters the Resource panel by "helm", moves to Helm Releases with the arrow keys
  and presses Enter
- **THEN** the Helm Releases list opens for the panel's context
