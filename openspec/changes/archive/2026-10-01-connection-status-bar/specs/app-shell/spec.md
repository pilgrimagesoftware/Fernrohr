## ADDED Requirements

### Requirement: Window status bar

Every workspace window SHALL show a status bar along its bottom edge with one item per cluster
context the window uses. Each item SHALL show the context name and
its connection state, and for any state other than connected, how long it has been in that state.
Items in a non-connected state SHALL be listed before connected ones.

#### Scenario: One item per cluster in the window

- **WHEN** a window uses `greedygoat` and `carefulcrab`
- **THEN** its status bar shows one item for each, and no item for contexts only other windows use

#### Scenario: Problem items first

- **WHEN** `carefulcrab` is connected and `greedygoat` is paused
- **THEN** the `greedygoat` item is listed before the `carefulcrab` item

#### Scenario: Closing panels keeps the item

- **WHEN** the user closes the last panel for a context the window still uses
- **THEN** that context's item stays in the window's status bar

### Requirement: Status severity is visually distinct

The status bar SHALL color each item by severity using the theme's semantic colors, and SHALL pair
every state with its own icon and text so the state can be read without color:

- connected: muted
- waiting for tunnel, or refreshing credentials: info
- tunnel reconnecting: warning
- connection failed, or any pause lasting longer than 30 seconds: danger

#### Scenario: Reconnecting draws attention

- **WHEN** a context's tunnel drops and its connection pauses to reconnect
- **THEN** its status bar item changes to the warning color with a reconnecting icon and the text "Reconnecting", and its elapsed time counts up each second

#### Scenario: Long pause escalates

- **WHEN** a pause has lasted more than 30 seconds
- **THEN** the item changes to the danger color while keeping its reason text

#### Scenario: Recovery returns to muted

- **WHEN** the paused connection resumes
- **THEN** the item returns to the muted connected state and its elapsed time is no longer shown

#### Scenario: Readable without color

- **WHEN** the status bar is viewed without color, for example in a grayscale screenshot
- **THEN** each item's state is identifiable from its icon and text alone
