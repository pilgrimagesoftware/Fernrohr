## Purpose

The application window frame: a docked, resizable panel workspace that can span multiple OS windows,
with window geometry and open panels restored on the next launch.

## Requirements

### Requirement: Main window with docked panel workspace

The application SHALL present a main window containing a panel workspace where panels can be arranged
by docking and splitting, and resized by dragging their separators.

#### Scenario: First launch shows an empty workspace

- **WHEN** the application starts with no saved workspace state
- **THEN** a single main window opens showing the cluster picker (see `cluster-picker`)
  rather than an empty panel area

#### Scenario: Panels can be split and resized

- **WHEN** the user adds a second panel to a workspace that already has one
- **THEN** the workspace shows both panels in a split arrangement
- **AND** dragging the separator between them changes their relative sizes

#### Scenario: Panels can be closed

- **WHEN** the user closes a panel
- **THEN** the panel is removed and the remaining panels reflow to fill the space

#### Scenario: Closing the last panel keeps the window connected

- **WHEN** the user closes a window's last remaining panel
- **THEN** the window stays connected to its cluster contexts and shows its Resource panel beside
  an empty panel area, not the cluster picker
- **AND** the window returns to the picker only when the user disconnects its last context

#### Scenario: Saved layout is restored per cluster

- **WHEN** a window connects to a cluster it has a saved layout for
- **THEN** that layout is restored instead of showing the full Resource panel

#### Scenario: No saved layout for this cluster shows the Resource panel

- **WHEN** a window connects to a cluster it has no saved layout for
- **THEN** the workspace shows the Resource panel listing that cluster's discovered
  resources (see `resource-browser`) instead of restoring an unrelated layout

### Requirement: Multiple windows

The application SHALL allow the user to open additional windows, each with an independent panel
workspace, and SHALL keep running while at least one window is open.

#### Scenario: Open a new window

- **WHEN** the user invokes the "new window" action
- **THEN** a second window opens with its own empty panel workspace
- **AND** panels in one window are unaffected by changes in the other

#### Scenario: Closing the last window exits

- **WHEN** the user closes the only remaining window
- **THEN** the application exits cleanly, persisting current state first

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

### Requirement: Window status bar

Every workspace window SHALL show a status bar along its bottom edge with one item per cluster
context the window uses. Each item SHALL show the context name and
its connection state, and for any state other than connected, how long it has been in that state.
Items in a non-connected state SHALL be listed before connected ones.

#### Scenario: One item per cluster in the window

- **WHEN** a window uses `cluster-a` and `cluster-b`
- **THEN** its status bar shows one item for each, and no item for contexts only other windows use

#### Scenario: Problem items first

- **WHEN** `cluster-b` is connected and `cluster-a` is paused
- **THEN** the `cluster-a` item is listed before the `cluster-b` item

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

### Requirement: The active panel tab holds keyboard focus
Whenever a docked panel tab becomes active - by clicking its tab, clicking inside its content,
keyboard tab navigation, opening a new panel, restoring a layout, or closing the tab that was
focused - that panel SHALL receive keyboard focus at once, so its keys work without a further
click or tab switch. Keyboard focus SHALL NOT be left on no panel while any panel is open.

#### Scenario: A tab accepts focus the first time it is clicked
- **WHEN** the user clicks a panel tab that has not had focus since the window opened
- **THEN** that panel has keyboard focus and its keys work immediately, without first switching
  to another tab and back

#### Scenario: A newly opened panel is focused
- **WHEN** the user opens a panel from the Resource panel or the command palette
- **THEN** the new panel's tab is active and the panel has keyboard focus

#### Scenario: Closing the focused tab transfers focus
- **WHEN** the user closes the panel tab that has keyboard focus and other panels remain open
- **THEN** focus moves to the tab that becomes active in its place, or to another open panel if
  its tab group is now empty, and is never lost

#### Scenario: Clicking a tab after a close gives it focus
- **WHEN** a tab has just been closed and the user clicks another panel tab
- **THEN** that tab's panel receives keyboard focus and keyboard input works immediately

### Requirement: Resource panel placement

The Resource panel SHALL be anchored to either edge of the main window, defaulting to a
user preference, movable to the other edge at runtime, and collapsible to reclaim window
space.

#### Scenario: Default anchor follows preference

- **WHEN** a window opens with a workspace
- **THEN** the Resource panel appears on the edge set by the user's preference

#### Scenario: Moving the Resource panel at runtime

- **WHEN** the user moves the Resource panel to the opposite edge
- **THEN** it re-anchors there for that window without affecting the user's saved
  preference unless the user also updates that preference

#### Scenario: Collapsing the Resource panel

- **WHEN** the user collapses the Resource panel
- **THEN** the workspace reclaims that space and other panels are unaffected

### Requirement: Panel focus

Exactly one dockable panel in a workspace SHALL hold focus at a time, be visually
distinguished as focused, and receive panel-scoped keyboard shortcuts.

#### Scenario: Clicking a panel focuses it

- **WHEN** the user clicks a dockable panel that does not have focus
- **THEN** that panel becomes focused, is visually distinguished from unfocused panels,
  and the previously focused panel loses that distinction

#### Scenario: Keyboard shortcuts target the focused panel

- **WHEN** a panel-scoped keyboard shortcut is invoked
- **THEN** it is dispatched to the currently focused panel, not any other open panel

### Requirement: Panel maximize

A dockable panel other than the Resource panel SHALL be maximizable to fill the
workspace area, excluding the Resource panel's space, with at most one panel maximized
per window at a time.

#### Scenario: Maximizing a panel

- **WHEN** the user maximizes a dockable panel
- **THEN** that panel fills the workspace area other than the Resource panel, and any
  other open panels are hidden until it is restored

#### Scenario: Maximizing a second panel restores the first

- **WHEN** the user maximizes a panel while another panel is already maximized
- **THEN** the previously maximized panel returns to its prior position and the newly
  maximized panel fills the workspace instead

### Requirement: Keyboard focus navigation between panels

The application SHALL let the user move keyboard focus between the visible panels of a window
from the keyboard, as registered commands offered in the command palette, bound to default keys
the user's keymap can override, and listed in the application menu. "Next" and "previous" SHALL
follow one stable order over the window's visible panels, wrapping from the last panel to the
first and from the first to the last. Panels the user cannot see - in a collapsed dock, or behind
another panel that is zoomed - SHALL NOT receive focus this way.

#### Scenario: Focus moves to the next panel

- **WHEN** a window shows two or more panels and one of them has keyboard focus
- **AND** the user invokes "focus next panel"
- **THEN** keyboard focus moves to the next panel in the order
- **AND** that panel's tab shows it has focus

#### Scenario: Focus wraps around

- **WHEN** the last panel in the order has keyboard focus
- **AND** the user invokes "focus next panel"
- **THEN** keyboard focus moves to the first panel
- **AND** invoking "focus previous panel" from the first panel moves focus to the last

#### Scenario: Previous reverses next

- **WHEN** the user invokes "focus next panel" and then "focus previous panel"
- **THEN** keyboard focus is back on the panel it started on

#### Scenario: No panel has focus yet

- **WHEN** no panel in the window has keyboard focus
- **AND** the user invokes "focus next panel"
- **THEN** keyboard focus moves to the first panel in the order

#### Scenario: Hidden panels are skipped

- **WHEN** a dock is collapsed, or another panel is zoomed
- **AND** the user moves focus with "focus next panel" or "focus previous panel"
- **THEN** focus only ever lands on panels that are visible

#### Scenario: Only one panel

- **WHEN** a window shows exactly one panel
- **AND** the user invokes "focus next panel" or "focus previous panel"
- **THEN** that panel has keyboard focus and nothing else changes

#### Scenario: Available from the command palette

- **WHEN** the user opens the command palette in a window with panels
- **THEN** "focus next panel" and "focus previous panel" are offered, each showing its key binding

### Requirement: An opened panel takes focus

When the user opens a panel, or shows one that is already open, the application SHALL move
keyboard focus to that panel, whichever route opened it (a click, a command, or a panel's own
shortcut).

#### Scenario: Opening a panel from the keyboard focuses it

- **WHEN** the user opens a pod's detail panel from a Pods panel with its keyboard shortcut
- **THEN** the pod detail panel has keyboard focus
- **AND** its own shortcuts respond without the user clicking it first

#### Scenario: Re-showing an open panel focuses it

- **WHEN** the user asks for a panel that is already open in the window
- **THEN** that panel is brought to the front of its tab group and has keyboard focus

### Requirement: Closing a panel closes the panel the user pointed at
Every docked panel tab SHALL carry its own close control that closes that tab's panel. Any close
control drawn for a tab group SHALL close that group's active panel. No close control SHALL close
a panel in a different tab group, whichever panel has focus.

#### Scenario: Closing the unfocused group's panel
- **WHEN** two tab groups are stacked, the upper group's Pods panel has focus, and the user clicks
  the close control of the lower group's RoleBindings tab
- **THEN** RoleBindings closes and Pods stays open with focus

#### Scenario: Closing a background tab
- **WHEN** a tab group holds Pods (active) and Services, and the user clicks the close control on
  the Services tab
- **THEN** Services closes and Pods stays the active tab

### Requirement: The last panel can be closed
Closing the last open panel in a window SHALL close it like any other. The window SHALL stay
connected to its cluster contexts, showing its Resource panel beside an empty panel area that
names the key to open a kind. Keyboard focus SHALL move to the Resource panel, expanding it if it
was collapsed.

#### Scenario: Closing the only panel
- **WHEN** a window has exactly one open panel and the user clicks its close control or presses
  `Cmd-W`
- **THEN** the panel closes, the window stays connected, and the Resource panel has focus

### Requirement: A lone panel keeps its close control beside its title
A tab group holding a single panel SHALL still show that panel's close control immediately beside
its title, where the tab's own close control sits when the group has several, not at the far edge
of the group.

#### Scenario: Closing down to one tab
- **WHEN** a tab group with two panels has one closed, leaving one
- **THEN** the remaining panel's close control is still drawn next to its title
