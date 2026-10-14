# selector-logs Specification

## Purpose
Following the logs of every pod a label selector picks, across a cluster context's namespaces, in
one Logs panel the user opens empty and types the selector into, rather than one pod at a time.

## Requirements

### Requirement: A Logs panel can open empty and follow a label selector
The application SHALL let the user open a Logs panel with no pod chosen, from the command palette
and from a menu item, and type a label selector into it. The panel SHALL stream every started
container of the pods that selector picks, with the same follow and scrollback as a single pod's
Logs panel. There SHALL be one such panel per cluster context; opening it again focuses it.

#### Scenario: Opened empty
- **WHEN** the user opens the label Logs panel
- **THEN** it shows an empty selector field with focus and streams nothing

#### Scenario: Applying a selector
- **WHEN** the user types `app=web` and presses Enter
- **THEN** the panel streams the logs of every started container of pods labelled `app=web` in its
  namespace scope, and states how many pods and containers matched

#### Scenario: Reopening focuses the panel
- **WHEN** the label Logs panel is open and the user opens it again
- **THEN** the existing panel is focused and no second one opens

#### Scenario: Follow and scrollback
- **WHEN** lines arrive from several pods while follow is on, then the user scrolls up
- **THEN** the view stays on the newest line, then stops auto-scrolling until the user returns to
  the bottom or re-enables follow

### Requirement: The selector is validated before it applies
The panel SHALL accept `kubectl -l` selector syntax - `key`, `!key`, `key=value`, `key!=value`,
`key in (a,b)` and `key notin (a,b)`, comma-separated - and SHALL reject text that does not parse
or that selects every pod, naming the cause and keeping the previous selector streaming.

#### Scenario: Invalid selector
- **WHEN** the user submits `app in (web`
- **THEN** the panel shows what is wrong with that requirement and keeps streaming the previous
  selector's pods

#### Scenario: Empty selector
- **WHEN** the user submits an empty selector
- **THEN** the panel says a selector needs a requirement and streams nothing new

#### Scenario: Set-based requirement
- **WHEN** the user submits `tier in (front,back),env!=dev`
- **THEN** only pods matching both requirements stream

### Requirement: Matching pods are followed as they come and go
The panel SHALL keep its streams in step with the cluster context's shared Pods watch: a pod that
begins matching, or whose container starts, SHALL start streaming, and a pod that is deleted or
stops matching SHALL stop. A recreated pod or restarted container SHALL stream afresh. The panel
SHALL NOT open a watch of its own.

#### Scenario: A pod appears later
- **WHEN** a pod matching the selector is created after the selector was applied
- **THEN** its logs begin streaming into the panel once a container has started

#### Scenario: A pod is deleted
- **WHEN** a streaming pod is deleted
- **THEN** its stream stops, its earlier lines stay, and the matched count drops

#### Scenario: A container restarts
- **WHEN** a streaming container restarts
- **THEN** its new instance's log streams, and the old instance's lines stay

#### Scenario: Closing the panel releases the watch
- **WHEN** the panel is closed
- **THEN** its streams stop and its share of the Pods watch is released

### Requirement: Concurrent streams are capped
The panel SHALL stream at most 20 containers at once, in a stable order, and SHALL say how many
containers matched and that only 20 stream when more match. Each container SHALL start at its last
100 lines.

#### Scenario: More matches than the cap
- **WHEN** a selector matches 25 containers
- **THEN** 20 stream and the panel states that 25 matched and 20 are streaming

#### Scenario: Tail window
- **WHEN** a matching container has 10,000 lines of history
- **THEN** the panel starts with that container's last 100 lines, then appends new ones

### Requirement: Each line shows its pod, in a stable colour
Every line SHALL be prefixed with its pod's name, and the container's name too when that pod has
several started containers. The prefix SHALL be coloured by a function of the pod's name alone, so
a pod keeps its colour across restarts of its container, re-syncs, and reopened panels. Containers
of one pod SHALL share its colour.

#### Scenario: Pods are told apart
- **WHEN** two matching pods emit lines
- **THEN** each line begins with its pod's name, in that pod's colour

#### Scenario: Colour is stable
- **WHEN** a pod's container restarts or the panel is restored from a saved layout
- **THEN** the pod's lines have the same colour as before

#### Scenario: Multi-container pod
- **WHEN** a pod with containers `app` and `sidecar` both stream
- **THEN** lines are prefixed `pod/app` and `pod/sidecar`, in one colour

### Requirement: Streams interleave in arrival order, and one ending does not end the panel
The panel SHALL append lines from all streams in the order they arrive. A stream that ends or
fails SHALL appear as a line tagged with its pod, without ending the panel or its other streams.

#### Scenario: Interleaving
- **WHEN** two pods emit lines alternately
- **THEN** the panel shows them in the order received

#### Scenario: One stream fails
- **WHEN** one container's log request fails
- **THEN** a line tagged with its pod gives the reason, and other pods keep streaming

### Requirement: The container filter narrows every matching pod
The panel SHALL let the user choose all containers (the default) or one named container, applied
to every matching pod. Containers the filter excludes SHALL NOT stream or count toward the cap.

#### Scenario: One container
- **WHEN** the user picks `sidecar` among the matched pods' containers
- **THEN** only containers named `sidecar` stream, and the prefix is the pod's name alone

#### Scenario: Back to all
- **WHEN** the user picks all containers
- **THEN** every started container of the matching pods streams again

### Requirement: The panel's namespace scope is visible and editable
The panel SHALL show the namespaces it searches - the namespace(s) it was opened in, a named
namespace set, or all namespaces - and let the user change that scope without leaving the panel.
Changing it SHALL stop streams of pods outside it and start those inside it.

#### Scenario: Scoped to the current namespace
- **WHEN** the panel opens from a panel scoped to `team-a`
- **THEN** only pods in `team-a` stream

#### Scenario: Switching to a set
- **WHEN** the user switches the panel to a namespace set naming `team-a` and `team-b`
- **THEN** pods in both namespaces stream, and pods elsewhere do not

#### Scenario: All namespaces
- **WHEN** the user chooses all namespaces
- **THEN** pods in any namespace may stream, subject to the cap

### Requirement: The panel is kept in saved layouts
A label Logs panel SHALL be restored from a saved layout with its selector, namespace scope and
container filter. A restored panel with a selector SHALL resume streaming once connected; one
without SHALL open empty.

#### Scenario: Restored with a selector
- **WHEN** a layout holding a panel following `app=web` is loaded
- **THEN** the panel shows that selector and streams its pods once connected

#### Scenario: Restored empty
- **WHEN** a layout holding a label Logs panel with no selector is loaded
- **THEN** it opens empty with the selector field focused

### Requirement: The typed panel's context is visible and can be changed
When its window holds more than one cluster context, the typed label Logs panel SHALL show the
context it searches beside its namespace scope. The user SHALL be able to switch it to another of
the window's contexts by mouse and by keyboard, with a command in the command palette that has a
default key and can be rebound. A panel in a window with one context SHALL show no context control.

#### Scenario: The context is shown with several contexts
- **WHEN** the typed panel is open in a window holding contexts `cluster-a` and `cluster-b`, and it
  searches `cluster-a`
- **THEN** its bar shows `cluster-a` beside its namespace scope

#### Scenario: No context control with one context
- **WHEN** the typed panel is open in a window holding only `cluster-a`
- **THEN** its bar shows no context control

#### Scenario: Switching by keyboard
- **WHEN** the typed panel has focus and the user runs the command to pick its context, moves to
  `cluster-b` and presses Enter
- **THEN** the panel searches `cluster-b`

#### Scenario: Switching by mouse
- **WHEN** the user clicks the panel's context control and chooses `cluster-b`
- **THEN** the panel searches `cluster-b`

### Requirement: Switching context follows the new context's pods
Switching the typed panel to another context SHALL stop every stream from the old context and
stream the pods its selector picks in the new one, with the same selector and container filter.
The namespace scope SHALL reset to all namespaces, because namespaces don't carry over between
clusters. If the chosen context already has a typed panel, that panel SHALL be focused and this one
left unchanged, so there is still one typed panel per context.

#### Scenario: The selector carries over
- **WHEN** the panel follows `app=web` in `cluster-a`, scoped to `team-a`, and the user switches it
  to `cluster-b`
- **THEN** no stream from `cluster-a` remains, pods labelled `app=web` in any namespace of
  `cluster-b` stream, and the namespace scope shows all namespaces

#### Scenario: The chosen context already has a typed panel
- **WHEN** `cluster-b` already has a typed panel and the user switches the `cluster-a` panel to
  `cluster-b`
- **THEN** the `cluster-b` panel is focused, and the `cluster-a` panel still searches `cluster-a`

### Requirement: A switched panel is saved in its new context
A typed label Logs panel switched to another context SHALL be saved in that context, so a saved
layout restores it there. A saved panel whose context the window doesn't hold SHALL be handled as
any other saved panel is - as a placeholder naming the missing context.

#### Scenario: Restored in the context it was switched to
- **WHEN** the user switches the typed panel from `cluster-a` to `cluster-b`, saves the layout, and
  loads it into a window holding both contexts
- **THEN** the restored panel searches `cluster-b`

#### Scenario: Restored without its context
- **WHEN** a layout holding a typed panel that searched `cluster-b` is loaded into a window holding
  only `cluster-a`
- **THEN** the panel's place shows a placeholder naming `cluster-b`, as for any other saved panel
