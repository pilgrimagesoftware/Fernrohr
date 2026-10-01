## ADDED Requirements

### Requirement: Window context bar

Every workspace window SHALL show a context bar along its top edge, below the title bar, with one
chip per context the window uses. Each chip SHALL show the context name, the name of its bound
tunnel if any, and a health indicator matching that context's status bar severity.

#### Scenario: Chips for each context

- **WHEN** a window uses `cluster-a` (through `qa-bastion`) and `cluster-c` (direct)
- **THEN** its context bar shows a `cluster-a` chip naming `qa-bastion` and an `cluster-c` chip with no tunnel name

#### Scenario: Health mirrors the status bar

- **WHEN** `cluster-a`'s tunnel drops and the status bar shows it reconnecting
- **THEN** the `cluster-a` chip's health indicator shows the same warning severity

### Requirement: Add a context to a window

The context bar SHALL offer an add control that lists every kubeconfig context the window does not
already use, with each context's tunnel binding. Choosing one SHALL add it to the window, reusing
the existing connection if one is already open for that context, open a Pods panel for it, and make
it the Resource panel's selected cluster.

#### Scenario: Add a second context

- **WHEN** a window using `cluster-a` adds `cluster-b`
- **THEN** a `cluster-b` chip appears, a Pods panel for `cluster-b` opens in the same dock, and the Resource panel lists `cluster-b`'s kinds

#### Scenario: Already-used contexts are not offered

- **WHEN** the user opens the add control in a window using `cluster-a`
- **THEN** `cluster-a` is not in the list

#### Scenario: Reuse another window's connection

- **WHEN** a second window adds a context that the first window is already connected to
- **THEN** no second connection or tunnel is started, and both windows show live data from the one connection

#### Scenario: Add fails

- **WHEN** the chosen context cannot connect
- **THEN** the add popover shows the connection failure, no chip is added for that context, and no panel is opened for it

### Requirement: Disconnect a context from a window

Each chip SHALL offer a disconnect action. Before acting, the application SHALL ask for
confirmation, stating how many panels in this window will close and, if other windows still use
the context, that it stays connected there. On confirmation it SHALL close every panel in this
window that uses the context and remove the chip. If no context remains, the window SHALL return to
the cluster picker.

#### Scenario: Confirm and close panels

- **WHEN** the user disconnects `cluster-b`, which has three panels in this window, and confirms
- **THEN** those three panels close, the chip is removed, and panels for other contexts are unchanged

#### Scenario: Cancel

- **WHEN** the user cancels the confirmation
- **THEN** no panel closes and the chip remains

#### Scenario: Context used elsewhere

- **WHEN** the user disconnects `cluster-a` while another window also uses it
- **THEN** the confirmation says `cluster-a` stays connected in one other window, and after confirming the other window keeps its live `cluster-a` panels

#### Scenario: Last context

- **WHEN** the user disconnects the only context in a window and confirms
- **THEN** the window shows the cluster picker

## MODIFIED Requirements

### Requirement: Workspace persistence

The application SHALL persist, per window, the window's screen geometry, the contexts it uses, and
the set, arrangement, and configuration of its open panels, and SHALL restore them on the next
launch, reconnecting every context a window used.

#### Scenario: Layout restored on relaunch

- **WHEN** the user has two windows with several arranged panels and quits the application
- **AND** the application is launched again
- **THEN** both windows reopen at their previous size and position with the same panels arranged the same way

#### Scenario: Multi-context window restored

- **WHEN** a window used `cluster-a` and `cluster-b` with panels for both when the application quit
- **THEN** on relaunch that window reconnects both contexts, shows both chips, and restores both contexts' panels in their previous arrangement

#### Scenario: Earlier single-context state loads

- **WHEN** the saved workspace state was written by a version that stored one context per window
- **THEN** each window restores with that one context

#### Scenario: Corrupt or unreadable state falls back to defaults

- **WHEN** the saved workspace state file cannot be parsed
- **THEN** the application starts with a single default main window
- **AND** the unreadable file is left in place rather than overwritten
