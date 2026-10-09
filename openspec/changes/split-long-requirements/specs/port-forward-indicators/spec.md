# Spec Delta

## MODIFIED Requirements

### Requirement: Stopping a forward is confirmed

Stopping a port-forward SHALL ask for confirmation on every path that stops one:

- the detail strip's stop icon
- a container port's stop icon
- the Stop Port Forward command
- Manage Tunnels' Stop button

Cancelling SHALL leave the forward running.

#### Scenario: Cancel keeps the forward

- **WHEN** the user clicks a forward's stop icon and then cancels the confirmation
- **THEN** the forward keeps running and every indicator still shows it

#### Scenario: Confirm from Manage Tunnels

- **WHEN** the user clicks Stop on a port-forward in Manage Tunnels
- **THEN** the confirmation names the forward, and the forward stops only after the user confirms

## ADDED Requirements

### Requirement: The stop confirmation names the forward

The confirmation for stopping a port-forward SHALL name the pod or service, the local address and
the target port, with the names set apart from the surrounding text as other confirmations do.

#### Scenario: The forward is named

- **WHEN** the user stops a forward from a pod's local address to one of its ports
- **THEN** the confirmation names the pod, the local address, and the target port

### Requirement: The stop confirmation is keyboard-operable

The confirmation for stopping a port-forward SHALL be fully operable from the keyboard: Enter
confirms, Escape cancels, and Tab moves between the buttons. Each button SHALL show its key (`⏎` and
`esc`).

#### Scenario: Escape keeps the forward

- **WHEN** the stop confirmation is open and the user presses Escape
- **THEN** the confirmation closes and the forward keeps running

#### Scenario: Enter stops it

- **WHEN** the stop confirmation is open and the user presses Enter
- **THEN** the forward stops
