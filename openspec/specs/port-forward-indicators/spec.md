# port-forward-indicators Specification

## Purpose
Shows active Kubernetes port-forwards where the user is looking, on the pod or service row, in the
pod's detail panel and next to the container port, and lets them be started, copied and stopped
from there by mouse or keyboard.

## Requirements

### Requirement: Forward indicator on list rows

The Pods list and the Services list SHALL have a Forwards column. For a row whose object has one
or more active port-forwards, the column SHALL show a forward icon with the number of forwards,
and its tooltip SHALL list each forward as `127.0.0.1:<local port> → <target port>`. Rows without
forwards SHALL show nothing in the column. The column SHALL update as forwards start and stop,
without the user refreshing.

#### Scenario: Forward appears on its row

- **WHEN** the user starts a port-forward on a pod with `shift-f`
- **THEN** that pod's row shows the forward icon with a count of 1, and hovering it shows the local address and container port

#### Scenario: Forward stopped elsewhere

- **WHEN** the forward is stopped from Manage Tunnels
- **THEN** the pod's row no longer shows the indicator

### Requirement: Forward strip in the pod detail panel

While the panel's pod has active port-forwards, the pod detail panel SHALL show a strip above its
tabs listing each forward's local address and target port. Each forward SHALL have an icon button
that copies its local address and an icon button that stops it, each with a tooltip. The strip
SHALL disappear when the last forward stops.

#### Scenario: Stop from the detail panel

- **WHEN** the user clicks a forward's stop icon in the pod detail strip and confirms
- **THEN** that forward stops, its local port closes, and it disappears from the strip, the pod's row and Manage Tunnels

### Requirement: Forward controls on container ports

The Containers tab SHALL show, next to each declared container port, an icon button with a tooltip
that starts a port-forward of that port. For a port that is being forwarded, it SHALL show the
local address with copy and stop icon buttons instead.

#### Scenario: Start by mouse

- **WHEN** the user clicks the forward icon next to container port 8080
- **THEN** a forward of port 8080 starts with no port prompt, and the port shows its local address with copy and stop buttons

### Requirement: Keyboard parity for forward controls

Starting and stopping a forward SHALL be registered commands available in the Pods list and in the
pod detail panel, guarded so they never fire while a text field has focus. **Port Forward**
(`shift-f`) starts one, asking for the port when there are several. **Stop Port Forward** stops
one, asking which when the pod has several. Both SHALL appear in the panels' hint rows while they
apply.

#### Scenario: Stop by keyboard

- **WHEN** a pod with one active forward is selected in the Pods list and the user invokes Stop Port Forward
- **THEN** the stop confirmation opens for that forward, without asking which one first

#### Scenario: Several forwards

- **WHEN** the user invokes Stop Port Forward in the detail panel of a pod with two forwards
- **THEN** the application asks which forward to stop

### Requirement: Stopping a forward is confirmed

Stopping a port-forward SHALL ask for confirmation on every path that stops one:

- the detail strip's stop icon
- a container port's stop icon
- the Stop Port Forward command
- Manage Tunnels' Stop button

The confirmation SHALL name the pod or service, the local address and the target port, with the
names set apart from the surrounding text as other confirmations do. It SHALL be fully operable
from the keyboard: Enter confirms, Escape cancels, and Tab moves between the buttons. Each button
SHALL show its key (`⏎` and `esc`). Cancelling
SHALL leave the forward running.

#### Scenario: Cancel keeps the forward

- **WHEN** the user clicks a forward's stop icon and then cancels the confirmation
- **THEN** the forward keeps running and every indicator still shows it

#### Scenario: Confirm from Manage Tunnels

- **WHEN** the user clicks Stop on a port-forward in Manage Tunnels
- **THEN** the confirmation names the forward, and the forward stops only after the user confirms

### Requirement: Results are not shown as list-panel notices

Starting a forward SHALL NOT show a success notice in the list panel. The indicators above report
success. A failure to start SHALL be reported as a transient notification naming the pod, the port
and the reason.

#### Scenario: Failure

- **WHEN** a forward can't start because the pod declares no container ports
- **THEN** a notification says so, and no notice appears inside the Pods list
