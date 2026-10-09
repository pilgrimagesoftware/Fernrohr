# Spec Delta

## MODIFIED Requirements

### Requirement: Window status bar

Every workspace window SHALL show a status bar along its bottom edge with one capsule per cluster
context the window uses, followed by the add-context control. Each capsule SHALL show, in order,
the context name, the name of its bound tunnel if any in square brackets, and an icon for its
connection state, and for any state other than connected, how long it has been in that state. The
tunnel name and the elapsed time SHALL render smaller than the context name, in the data font and
the theme's muted foreground color. The state icon SHALL have a tooltip naming the state and, for
any state other than connected, how long it has lasted. Capsules in a non-connected state SHALL be listed
before connected ones.

#### Scenario: One item per cluster in the window

- **WHEN** a window uses `cluster-a` (through `qa-bastion`) and `cluster-b` (direct)
- **THEN** its status bar shows a `cluster-a` capsule reading `cluster-a [qa-bastion]` followed by
  its state icon, a `cluster-b` capsule with no tunnel name, and no capsule for contexts only other
  windows use

#### Scenario: State shown as an icon

- **WHEN** the user hovers the state icon of a connected `cluster-b` capsule
- **THEN** a tooltip reads "Connected", and the capsule itself shows no state text

#### Scenario: Problem items first

- **WHEN** `cluster-b` is connected and `cluster-a` is paused
- **THEN** the `cluster-a` capsule is listed before the `cluster-b` capsule

#### Scenario: Closing panels keeps the item

- **WHEN** the user closes the last panel for a context the window still uses
- **THEN** that context's capsule stays in the window's status bar

#### Scenario: Each context appears once

- **WHEN** a window uses one context
- **THEN** that context's name and health appear in exactly one place in the window chrome

### Requirement: Status severity is visually distinct

The status bar SHALL color each item by severity using the theme's semantic colors, and SHALL pair
every state with its own distinct icon, and its state text in the icon's tooltip, so the state can
be read without color:

- connected: muted
- waiting for tunnel, or refreshing credentials: info
- tunnel reconnecting: warning
- connection failed, or any pause lasting longer than 30 seconds: danger

#### Scenario: Reconnecting draws attention

- **WHEN** a context's tunnel drops and its connection pauses to reconnect
- **THEN** its status bar item changes to the warning color with a reconnecting icon whose tooltip reads "Reconnecting", and its elapsed time counts up each second

#### Scenario: Long pause escalates

- **WHEN** a pause has lasted more than 30 seconds
- **THEN** the item changes to the danger color while keeping its icon and its reason in the tooltip

#### Scenario: Recovery returns to muted

- **WHEN** the paused connection resumes
- **THEN** the item returns to the muted connected state and its elapsed time is no longer shown

#### Scenario: Readable without color

- **WHEN** the status bar is viewed without color, for example in a grayscale screenshot
- **THEN** each item's state is identifiable from its icon alone
