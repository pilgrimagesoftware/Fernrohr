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

### Requirement: Pod detail follows its pod's deletion live
While its detail panel is open, a pod's deletion SHALL be reflected live. Once the pod has a deletion
timestamp, the panel SHALL show it as Terminating with its remaining grace period. Once the pod is
gone, the panel SHALL say within a few seconds that it was deleted and when, SHALL keep showing its
last known state marked as stale rather than clearing it, SHALL keep its Events tab listing the pod's
events, and SHALL stay open until the user closes it. If a new pod then appears under the same name
(a different uid), the panel SHALL switch to the new pod and show that it replaced the deleted one.

#### Scenario: Deleted while open
- **WHEN** a pod's detail panel is open and the pod is deleted
- **THEN** the panel first shows Terminating, then says the pod was deleted and when, keeps its last
  known fields visible as stale, and still lists its events

#### Scenario: Recreated under the same name
- **WHEN** a StatefulSet pod's detail panel is open and the pod is deleted and recreated as `web-0`
- **THEN** the panel switches to the new `web-0`, shows that it replaced the deleted pod, and its
  fields follow the new pod live
