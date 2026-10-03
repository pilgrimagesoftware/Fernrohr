# Spec Delta

## ADDED Requirements

### Requirement: Object detail follows its object live
The object detail panel SHALL reflect its object's current state while it is open, updating its
sections, metadata and YAML within a few seconds of a change in the cluster, without the user
reopening or refreshing it, and keeping the user's view state.

#### Scenario: A Deployment rolls out
- **WHEN** a Deployment's detail panel is open and the Deployment is scaled from 2 to 3 replicas
- **THEN** the panel's desired, ready and available replica counts update as the rollout proceeds

#### Scenario: Deleted while open
- **WHEN** an object's detail panel is open and the object is deleted
- **THEN** within a few seconds the panel says it was deleted and when, keeps its last known state
  visible as stale, and stays open
