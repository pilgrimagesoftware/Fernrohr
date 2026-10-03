# Spec Delta

## Purpose

A panel for reading a cluster's retained events in bulk: live, filterable by type, namespace, kind
and reason, searchable, and linked to the objects the events are about.

## ADDED Requirements

### Requirement: An events browser panel
The application SHALL open an events browser panel for a cluster context from the Event kind in the
Resource panel and from a registered "Events" command. The panel SHALL list every event the cluster
retains that the user may list, newest first by last seen time, showing last seen, type, reason,
involved object, message, count and source, and SHALL update live as events are created, updated or
expire.

#### Scenario: Opening from the Resource panel
- **WHEN** the user opens Events from the Resource panel
- **THEN** the events browser opens for that context, not the generic list

#### Scenario: Live updates
- **WHEN** the events browser is open and a deployment is scaled, producing new events
- **THEN** those events appear at the top within a few seconds

#### Scenario: Default sort is newest first
- **WHEN** an events browser opens with no saved sort
- **THEN** it is sorted by age ascending (newest first), with the age column's header showing that
  sort, and the user can sort by any other column; a changed sort is saved with the panel

#### Scenario: Both event APIs read
- **WHEN** an event was recorded only through `events.k8s.io/v1` fields
- **THEN** it shows its last seen time, reason and message like any other

### Requirement: Events can be filtered
The events browser SHALL offer filters for type (Warning, Normal), involved object kind and reason,
each a multi-select whose options are the values present in the current events, combined with the
panel's namespace scope. Active filters SHALL be shown on the panel and clearable individually and
all at once, and SHALL persist with the panel.

#### Scenario: Warnings only
- **WHEN** the user selects type Warning
- **THEN** only Warning events remain, and the filter shows as active

#### Scenario: Combining filters
- **WHEN** the user selects kind Pod and reason BackOff in namespace `payments`
- **THEN** only BackOff events about Pods in `payments` remain

#### Scenario: Restored with the window
- **WHEN** the application restarts with a filtered events browser in a saved layout
- **THEN** it reopens with the same filters

### Requirement: Events can be searched
The events browser SHALL provide a search box that matches the visible columns, including the
message, by default, using the same search box and behaviour as other list panels.

#### Scenario: Searching messages
- **WHEN** the user searches for `ImagePullBackOff`
- **THEN** only events whose visible columns contain it remain, with a match count

### Requirement: Events lead to their objects
The involved object of every event SHALL be a link that opens that object's detail panel, and the
selected event's full message SHALL be shown untruncated.

#### Scenario: Following an event to its pod
- **WHEN** the user selects a BackOff event and follows its involved object
- **THEN** that Pod's detail panel opens

#### Scenario: Reading a long message
- **WHEN** an event's message is longer than its column
- **THEN** selecting the event shows the whole message, selectable and copyable

### Requirement: The events browser is keyboard-operable
Every action in the events browser - moving the selection, opening the involved object, focusing the
search box, and setting and clearing each filter - SHALL be reachable from the keyboard and listed in
the panel's hint row.

#### Scenario: Filter by keyboard
- **WHEN** the events browser has focus and the user presses the type-filter key and chooses Warning
- **THEN** only Warning events remain, without using the mouse
