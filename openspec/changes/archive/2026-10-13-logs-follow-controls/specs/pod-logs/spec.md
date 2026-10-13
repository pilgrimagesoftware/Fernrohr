# Spec Delta

## MODIFIED Requirements

### Requirement: Follow mode

The log panel SHALL support a follow mode that keeps the newest line in view, and SHALL suspend
following when the user scrolls up and resume it when the user returns to the bottom. The panel
SHALL show whether follow is on or off with a visible indication on the Follow control, and SHALL
scroll to the newest line when follow is turned on, if the view is not already there.

#### Scenario: Following the tail

- **WHEN** follow mode is on and new lines arrive
- **THEN** the view stays pinned to the newest line

#### Scenario: Scrolling up pauses follow

- **WHEN** the user scrolls up while following
- **THEN** the view stops auto-scrolling until the user scrolls back to the bottom or re-enables follow

## ADDED Requirements

### Requirement: Follow state is visible
The Follow control SHALL be visibly distinct when follow is on and when it is off, and SHALL change
when scrolling suspends or resumes follow.

#### Scenario: Follow is on
- **WHEN** follow mode is on
- **THEN** the Follow control is shown in its active style

#### Scenario: Follow is off
- **WHEN** follow mode is off, including after the user scrolls up
- **THEN** the Follow control is shown in its plain style

### Requirement: Turning follow on scrolls to the bottom
Turning follow on SHALL scroll the log view to the newest line immediately when it is not already
there, without waiting for a new line to arrive.

#### Scenario: Re-enabling follow after scrolling up
- **WHEN** the user has scrolled up and turns follow on
- **THEN** the view scrolls to the newest line at once

#### Scenario: Already at the bottom
- **WHEN** the view is already at the newest line and the user turns follow on
- **THEN** the view does not move
