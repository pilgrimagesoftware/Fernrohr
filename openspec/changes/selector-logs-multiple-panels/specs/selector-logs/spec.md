# Spec Delta

## ADDED Requirements

### Requirement: Label Logs panels open empty, as often as asked, and follow a label selector
The application SHALL let the user open a Logs panel with no pod chosen, from the command palette
and from a menu item, and type a label selector into it. The panel SHALL stream every started
container of the pods that selector picks, with the same follow and scrollback as a single pod's
Logs panel. Opening it SHALL always open a new, empty panel, so several can be open at once in the
same cluster context or different ones, each with its own state.

#### Scenario: Opened empty
- **WHEN** the user opens the label Logs panel
- **THEN** it shows an empty selector field with focus and streams nothing

#### Scenario: Applying a selector
- **WHEN** the user types `app=web` and presses Enter
- **THEN** the panel streams the logs of every started container of pods labelled `app=web` in its
  namespace scope, and states how many pods and containers matched

#### Scenario: Opening another panel
- **WHEN** a label Logs panel following `app=web` is open and the user opens the label Logs panel
  again in the same context
- **THEN** a second, empty panel opens with its selector field focused, and the first keeps
  streaming `app=web`

#### Scenario: Two panels stream independently
- **WHEN** one label Logs panel follows `app=web` and another, in the same context, follows
  `app=worker`
- **THEN** each streams only its own selector's pods, and each panel's title names its selector

#### Scenario: Follow and scrollback
- **WHEN** lines arrive from several pods while follow is on, then the user scrolls up
- **THEN** the view stays on the newest line, then stops auto-scrolling until the user returns to
  the bottom or re-enables follow

## MODIFIED Requirements

### Requirement: The panel is kept in saved layouts
A label Logs panel SHALL be restored from a saved layout with its selector, namespace scope and
container filter. A restored panel with a selector SHALL resume streaming once connected; one
without SHALL open empty. A layout holding several label Logs panels SHALL restore each of them
separately, each with its own state. A layout saved when only one typed panel per context was
allowed SHALL still restore.

#### Scenario: Restored with a selector
- **WHEN** a layout holding a panel following `app=web` is loaded
- **THEN** the panel shows that selector and streams its pods once connected

#### Scenario: Restored empty
- **WHEN** a layout holding a label Logs panel with no selector is loaded
- **THEN** it opens empty with the selector field focused

#### Scenario: Several panels restored
- **WHEN** a layout holding two label Logs panels in one context, following `app=web` and
  `app=worker`, is loaded
- **THEN** both panels are restored, each with its own selector and streams

### Requirement: Switching context follows the new context's pods
Switching the typed panel to another context SHALL stop every stream from the old context and
stream the pods its selector picks in the new one, with the same selector and container filter.
The namespace scope SHALL reset to all namespaces, because namespaces don't carry over between
clusters. The panel itself SHALL move to the chosen context, in its own tab, even when another
typed panel already searches that context. Switching SHALL never focus a different panel.

#### Scenario: The selector carries over
- **WHEN** the panel follows `app=web` in `cluster-a`, scoped to `team-a`, and the user switches it
  to `cluster-b`
- **THEN** no stream from `cluster-a` remains, pods labelled `app=web` in any namespace of
  `cluster-b` stream, and the namespace scope shows all namespaces

#### Scenario: The chosen context already has a typed panel
- **WHEN** `cluster-b` already has a typed panel following `app=worker`, and the user switches the
  `cluster-a` panel following `app=web` to `cluster-b`
- **THEN** both panels search `cluster-b`, one following `app=web` in the switched panel's tab and
  the other still following `app=worker`, and focus stays on the switched panel

## REMOVED Requirements

### Requirement: A Logs panel can open empty and follow a label selector
**Reason**: It allowed one typed label Logs panel per cluster context and made opening it again
focus that panel. Users need several typed panels at once.
**Migration**: Replaced by "Label Logs panels open empty, as often as asked, and follow a label
selector", which keeps every other behavior and scenario of this requirement but opens a new panel
each time.
