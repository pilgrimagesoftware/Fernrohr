# Spec Delta

## ADDED Requirements

### Requirement: Pod detail follows its pod live
The pod detail panel SHALL reflect the pod's current state while it is open: when the pod's phase,
conditions, container states, restart counts or any other shown field change in the cluster, the
panel SHALL update within a few seconds, without the user reopening or refreshing it, keeping the
selected tab, scroll position and any revealed or expanded state.

#### Scenario: A pod recovers from a missing Secret
- **WHEN** a pod's detail panel shows it Pending because a referenced Secret is missing, and the
  Secret is then created and the pod starts
- **THEN** the panel shows the pod Running, with its containers ready, without being reopened

#### Scenario: A container restarts
- **WHEN** a container in the open pod restarts
- **THEN** its restart count and state update in place, and the user stays on the tab they were on
