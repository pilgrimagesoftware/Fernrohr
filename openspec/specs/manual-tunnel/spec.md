# manual-tunnel Specification

## Purpose
Lets a context depend on a network path the user must bring up by hand, such as a menu-bar VPN client, by holding the connection until the user confirms the path is up.

## Requirements

### Requirement: Manual tunnel settings
A manual tunnel SHALL carry an optional instruction message and a skip-when-reachable setting that is on by default. A manual tunnel SHALL NOT carry a command, host, port, or credentials, and SHALL NOT start any process.

#### Scenario: Create a manual tunnel
- **WHEN** the user creates a tunnel named `corp-vpn`, chooses the manual kind, and enters the message "Connect the corporate VPN in the menu bar"
- **THEN** the tunnel saves, is marked as a manual tunnel in the tunnel list, and is offered in every context's tunnel selector

### Requirement: Connection held for confirmation
When a context bound to a manual tunnel connects and the tunnel is not already confirmed, the application SHALL hold that connection in an awaiting-confirmation state until the user chooses Proceed or Cancel. The wait SHALL have no timeout.

#### Scenario: Proceed releases the connection
- **WHEN** the user connects a context bound to `corp-vpn`, brings up the VPN, and chooses Proceed
- **THEN** the connection leaves the awaiting-confirmation state and connects directly to the context's API server

#### Scenario: Unreachable after Proceed
- **WHEN** the user chooses Proceed but the API server is still unreachable
- **THEN** the connection fails with the usual connection error, the confirmation is withdrawn, and the next attempt prompts again

### Requirement: Waiting never blocks other work
While a connection awaits confirmation, the application SHALL keep its UI responsive and SHALL NOT delay, block, or disturb any other context's connection, session, or panel.

#### Scenario: Other contexts keep working
- **WHEN** one context is awaiting confirmation of `corp-vpn` and the user connects a different context with no tunnel
- **THEN** the other context connects immediately and its panels load while the first context keeps waiting

### Requirement: One confirmation per tunnel
All contexts bound to the same manual tunnel SHALL share one confirmation. Only one prompt SHALL be outstanding per manual tunnel, and Proceed or Cancel SHALL apply to every context waiting on it.

#### Scenario: Two contexts, one prompt
- **WHEN** two contexts bound to `corp-vpn` connect while it is unconfirmed
- **THEN** one prompt is shown, and a single Proceed lets both connections continue

#### Scenario: Already confirmed
- **WHEN** a second context bound to `corp-vpn` connects while another connected context is already using the confirmed tunnel
- **THEN** it connects without a prompt

### Requirement: Cancel fails waiting connections
Choosing Cancel SHALL fail every connection waiting on that manual tunnel with a reason stating the user cancelled it. It SHALL NOT fall back to a direct connection.

#### Scenario: User cancels
- **WHEN** the user chooses Cancel for `corp-vpn` while a context is waiting on it
- **THEN** that connection fails with a reason naming the tunnel and the cancellation, and no Kubernetes request is sent

### Requirement: Confirmation lifetime
A confirmation SHALL last while at least one connected context uses the manual tunnel. When the last such context disconnects, the confirmation SHALL be discarded and the next connection SHALL prompt again.

#### Scenario: Prompt again after full disconnect
- **WHEN** every context using `corp-vpn` disconnects and the user later connects one of them again
- **THEN** the user is prompted again

### Requirement: Skip when already reachable
When skip-when-reachable is on, before prompting the application SHALL try a short TCP connect to the context's API server host and port. If it succeeds, the application SHALL treat the tunnel as confirmed without prompting.

#### Scenario: VPN already up
- **WHEN** the user connects a context bound to `corp-vpn`, skip-when-reachable is on, and the API server accepts a TCP connection
- **THEN** the context connects without a prompt or notification

#### Scenario: Shortcut turned off
- **WHEN** skip-when-reachable is off for `corp-vpn`
- **THEN** every new confirmation prompts, even when the API server is reachable

### Requirement: Prompt surfaces
While a manual tunnel awaits confirmation, the application SHALL post one desktop notification and show a cluster-picker status line with Proceed and Cancel, each naming the tunnel and waiting contexts and showing the tunnel's message.

#### Scenario: User is in another app
- **WHEN** a context bound to `corp-vpn` starts waiting while Fernrohr is not the active application
- **THEN** a desktop notification names `corp-vpn` and shows its message

### Requirement: Awaiting-confirmation context capsule
While a context awaits confirmation, its status-bar capsule SHALL keep the standard capsule layout, drawn in an attention color distinct from every other connection state, with an awaiting-confirmation state icon whose tooltip names the state, the elapsed wait, and the tunnel's message. Its menu SHALL offer Proceed and Cancel above Disconnect.

#### Scenario: Capsule draws attention
- **WHEN** a context is waiting on `corp-vpn`
- **THEN** its capsule is drawn in the attention color with the tunnel name, its own state icon, and the elapsed time, and hovering the icon shows "Awaiting confirmation" with the tunnel's message

#### Scenario: Proceed from the capsule menu
- **WHEN** the user opens the waiting context's capsule menu and chooses Proceed
- **THEN** every connection waiting on `corp-vpn` proceeds, and the capsule returns to its normal connected appearance once connected

#### Scenario: Shared tunnel, two capsules
- **WHEN** two contexts in the window are waiting on `corp-vpn`
- **THEN** both capsules show the attention state, and Proceed or Cancel from either capsule's menu resolves both

### Requirement: Keyboard-operable confirmation
Proceed and Cancel SHALL be registered commands available from the command palette and keybindings while any manual tunnel awaits confirmation. When several tunnels are waiting, the command SHALL ask which tunnel it applies to.

#### Scenario: Proceed from the palette
- **WHEN** a context is waiting on `corp-vpn` and the user runs Proceed with Manual Tunnel from the command palette
- **THEN** the waiting connection proceeds without the user touching the mouse

#### Scenario: Several tunnels waiting
- **WHEN** `corp-vpn` and `lab-vpn` are both waiting and the user runs Proceed with Manual Tunnel
- **THEN** a picker lists both tunnels and only the chosen one proceeds
