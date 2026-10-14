# Spec Delta

## ADDED Requirements

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
