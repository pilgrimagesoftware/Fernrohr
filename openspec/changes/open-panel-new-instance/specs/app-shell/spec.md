# Spec Delta

## ADDED Requirements

### Requirement: Any panel can be opened again in a new panel
The application SHALL offer an Open in New Panel command, in the command palette and the View menu
with a default key that can be rebound. It opens another instance of the focused panel, with the
same resource, cluster context and namespace scope, as a new panel with keyboard focus, even though
that panel is already open. Requests that aren't explicitly for a new panel SHALL still focus an
open panel for the same resource.

#### Scenario: Duplicating the focused panel
- **WHEN** a Deployments list in `team-a` has focus and the user runs Open in New Panel
- **THEN** a second Deployments list scoped to `team-a` opens in its own tab with keyboard focus,
  and the first stays open

#### Scenario: Works for any panel kind
- **WHEN** a pod's detail panel, an object's detail panel, a Logs panel or the Events browser has
  focus and the user runs Open in New Panel
- **THEN** a second panel of the same kind on the same resource opens

#### Scenario: Default opens still focus
- **WHEN** two instances of a Service's detail panel are open and the user double-clicks that
  Service's row
- **THEN** the first instance opened is focused, and no third panel opens

#### Scenario: Instances are independent
- **WHEN** two instances of a Deployments list are open and the user changes one's namespace scope
- **THEN** the other keeps its own scope

### Requirement: Duplicate panels are told apart
The first panel open on a resource in a window SHALL be numbered 1. Each further instance SHALL
take the lowest number of 2 or more that no open panel on the same resource uses. An instance
numbered 2 or more SHALL show its number after its title and tab name, as ` (2)`. The first SHALL
show none.

#### Scenario: Numbered titles
- **WHEN** a pod's detail panel is open and the user opens two more instances of it
- **THEN** their titles end in ` (2)` and ` (3)`, and the first's title is unchanged

#### Scenario: A freed number is reused
- **WHEN** instances 1, 2 and 3 of a Deployments list are open, the user closes ` (2)`, then opens
  another instance
- **THEN** the new instance is numbered ` (2)`
