# Spec Delta

## ADDED Requirements

### Requirement: Window title names the window's cluster contexts

Every application window SHALL carry a title that identifies which cluster context, or contexts,
it is showing, and SHALL update that title whenever the set of contexts the window holds changes.

The title SHALL name the active cluster context while the window shows exactly one, and SHALL
state how many contexts the window holds when it shows more than one. A window that is not yet
connected to a cluster SHALL be titled with the application name alone.

#### Scenario: A connected window names its cluster

- **WHEN** the user connects a window to the cluster context named `staging`
- **THEN** the window's title names `staging` alongside the application name
- **AND** the macOS Window menu lists the window under that same title

#### Scenario: A window with no cluster shows only the application name

- **WHEN** a window has not yet been connected to any cluster and is showing the cluster picker
- **THEN** the window's title is the application name with no cluster named

#### Scenario: A window holding several contexts says how many

- **WHEN** a window is connected to two or more cluster contexts
- **THEN** the window's title states the number of contexts the window is showing rather than naming a single one

#### Scenario: Adding a context re-titles the window

- **WHEN** the user adds a cluster context to a window that was showing one context
- **THEN** the window's title changes to reflect that it is now showing more than one

#### Scenario: Disconnecting a context re-titles the window

- **WHEN** the user disconnects one context from a window that was showing several
- **THEN** the window's title changes to name the context it is now showing

#### Scenario: Disconnecting the last context returns the window to the plain title

- **WHEN** the user disconnects the only context a window holds
- **THEN** the window returns to showing the cluster picker
- **AND** the window's title is the application name alone

#### Scenario: Restored windows come back titled

- **WHEN** a window holding several cluster contexts is closed and reopened on the next launch
- **THEN** the reopened window's title reflects the contexts it restored
