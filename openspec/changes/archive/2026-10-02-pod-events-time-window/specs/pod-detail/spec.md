# Spec Delta

## MODIFIED Requirements

### Requirement: The Events tab lists the pod's own events
The pod detail panel SHALL show, in an Events tab, the events whose involved object is this pod,
newest first, and SHALL report a failure to list them rather than showing an empty list. The list
SHALL update while the panel is open as events are created, updated or expire, and SHALL show only
events last seen within the panel's time window: 15 minutes, 1 hour, 6 hours, 24 hours, or All.
When the window hides events, the tab SHALL say how many.

#### Scenario: Events are this pod's, newest first
- **WHEN** the pod has events, including ones recorded only through `events.k8s.io/v1` fields
- **THEN** the Events tab lists them newest first with their age and repeat count, and lists no
  event belonging to another object of the same name

#### Scenario: Events cannot be listed
- **WHEN** the pod can be read but listing its events fails (for example, RBAC forbids it)
- **THEN** the pod's fields still load, and the Events tab says the events could not be listed

#### Scenario: A new event appears live
- **WHEN** the Events tab is open and the pod is restarted, producing a new `BackOff` event
- **THEN** the event appears at the top of the list within a few seconds, without reopening the
  panel

#### Scenario: Narrowing the window
- **WHEN** the window is 1 hour and the user switches it to 15 minutes
- **THEN** only events last seen in the past 15 minutes remain, and the tab says how many older
  events are hidden

#### Scenario: Events age out of the window
- **WHEN** an event's last occurrence passes the edge of the window while the tab is open
- **THEN** it leaves the list without the user refreshing

## ADDED Requirements

### Requirement: The events window is a remembered preference
The time window chosen in a pod detail panel SHALL become the default for pod detail panels opened
afterwards and SHALL persist across restarts. Changing it SHALL be reachable by keyboard and from the
command palette.

#### Scenario: Window survives a restart
- **WHEN** the user sets the window to 6 hours and restarts the application
- **THEN** a newly opened pod detail panel's Events tab uses 6 hours

### Requirement: The Overview surfaces recent warnings
The pod detail Overview tab SHALL show the pod's Warning events within the panel's time window,
newest first, limited to the most recent few, with a link to the Events tab, and SHALL show nothing
for events when there are none in the window.

#### Scenario: A crashing pod
- **WHEN** a pod has `BackOff` Warning events in the past 15 minutes and its detail panel opens with a
  1 hour window
- **THEN** the Overview tab shows those warnings without the user switching tabs, and following the
  link opens the Events tab
