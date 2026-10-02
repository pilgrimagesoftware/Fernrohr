# Spec Delta

## ADDED Requirements

### Requirement: Window toolbar
Every window SHALL draw its top bar as a gpui-kit Toolbar showing the application icon and name,
in place of a separate context bar.

#### Scenario: Toolbar contents
- **WHEN** a workspace window is open
- **THEN** its top bar shows the app icon and the app name, and no cluster context chips

### Requirement: Theme switcher in the status bar
The status bar SHALL show a theme switcher at the end opposite its context capsules, offering
System, Light and Dark. Choosing one SHALL apply it to every window at once and SHALL persist it as
the theme preference. The switcher SHALL also be a registered command reachable from the command
palette.

#### Scenario: Switching to dark
- **WHEN** the user picks Dark from the status bar's theme switcher
- **THEN** every open window redraws in the dark theme, and the next launch starts in dark

#### Scenario: Following the system
- **WHEN** the theme is System and the OS switches from light to dark appearance
- **THEN** the application follows without restarting

## MODIFIED Requirements

### Requirement: Window status bar

Every workspace window SHALL show a status bar along its bottom edge with one capsule per cluster
context the window uses, followed by the add-context control. Each capsule SHALL show the context
name, the name of its bound tunnel if any, and its connection state, and for any state other than
connected, how long it has been in that state. Capsules in a non-connected state SHALL be listed
before connected ones.

#### Scenario: One item per cluster in the window

- **WHEN** a window uses `cluster-a` (through `qa-bastion`) and `cluster-b` (direct)
- **THEN** its status bar shows a `cluster-a` capsule naming `qa-bastion` and a `cluster-b` capsule
  with no tunnel name, and no capsule for contexts only other windows use

#### Scenario: Problem items first

- **WHEN** `cluster-b` is connected and `cluster-a` is paused
- **THEN** the `cluster-a` capsule is listed before the `cluster-b` capsule

#### Scenario: Closing panels keeps the item

- **WHEN** the user closes the last panel for a context the window still uses
- **THEN** that context's capsule stays in the window's status bar

#### Scenario: Each context appears once

- **WHEN** a window uses one context
- **THEN** that context's name and health appear in exactly one place in the window chrome

### Requirement: Add a context to a window

The status bar SHALL offer an add control after its capsules that lists every kubeconfig context
the window does not already use, with each context's tunnel binding. Choosing one SHALL add it to
the window, reusing the existing connection if one is already open for that context, open a Pods
panel for it, and make it the Resource panel's selected cluster.

#### Scenario: Add a second context

- **WHEN** a window using `cluster-a` adds `cluster-b`
- **THEN** a `cluster-b` capsule appears, a Pods panel for `cluster-b` opens in the same dock, and
  the Resource panel lists `cluster-b`'s kinds

#### Scenario: Already-used contexts are not offered

- **WHEN** the user opens the add control in a window using `cluster-a`
- **THEN** `cluster-a` is not in the list

#### Scenario: Reuse another window's connection

- **WHEN** a second window adds a context that the first window is already connected to
- **THEN** no second connection or tunnel is started, and both windows show live data from the one connection

#### Scenario: Add fails

- **WHEN** the chosen context cannot connect
- **THEN** the add popover shows the connection failure, no capsule is added for that context, and
  no panel is opened for it

### Requirement: Disconnect a context from a window

Each status bar capsule SHALL offer a disconnect action. Before acting, the application SHALL ask
for confirmation, stating how many panels in this window will close and, if other windows still use
the context, that it stays connected there. On confirmation it SHALL close every panel in this
window that uses the context and remove the capsule. If no context remains, the window SHALL return
to the cluster picker.

#### Scenario: Confirm and close panels

- **WHEN** the user disconnects `cluster-b`, which has three panels in this window, and confirms
- **THEN** those three panels close, the capsule is removed, and panels for other contexts are unchanged

#### Scenario: Cancel

- **WHEN** the user cancels the confirmation
- **THEN** no panel closes and the capsule remains

#### Scenario: Context used elsewhere

- **WHEN** the user disconnects `cluster-a` while another window also uses it
- **THEN** the confirmation says `cluster-a` stays connected in one other window, and after confirming the other window keeps its live `cluster-a` panels

#### Scenario: Last context

- **WHEN** the user disconnects the only context in a window and confirms
- **THEN** the window shows the cluster picker

## REMOVED Requirements

### Requirement: Window context bar
**Reason**: Its chips duplicated the status bar's per-context items; the two are merged into the
status bar's capsules.
**Migration**: Context name, tunnel name and health are on each status bar capsule; add and
disconnect moved with them.
