# Spec Delta

## Purpose

Lets a user pin individual Kubernetes objects for one-step return, and lists every pinned object
in a dedicated panel, across clusters and independent of any one window.

## ADDED Requirements

### Requirement: Pinning an object
The user SHALL be able to pin and unpin any object that has a detail panel, identified by its
cluster context, kind, namespace (if namespaced), and name. Pinning SHALL NOT require that
object's panel to be the one currently focused, only that the object be identified unambiguously
(its own open panel, or a `resource-links` reference to it).

#### Scenario: Pin from the object's own panel
- **WHEN** a Deployment's detail panel is open and the user pins it
- **THEN** that Deployment appears in the Favorites panel

#### Scenario: Unpin removes it
- **WHEN** a pinned object is unpinned
- **THEN** it no longer appears in the Favorites panel

#### Scenario: Pinning is idempotent
- **WHEN** the user pins an object that is already pinned
- **THEN** it still appears exactly once in the Favorites panel

### Requirement: Pins persist and are not window-scoped
Pinned objects SHALL survive an application restart, and a pin made in one window SHALL be
visible from every other open window immediately.

#### Scenario: Pin survives a restart
- **WHEN** the user pins an object and restarts the application
- **THEN** the Favorites panel lists that object on the next launch

#### Scenario: Pin is visible across windows
- **WHEN** two windows are open and the user pins an object from one
- **THEN** the other window's Favorites panel, if open, shows the new pin without that window
  being reopened

### Requirement: Favorites panel
The application SHALL provide a dockable Favorites panel listing every pinned object, grouped by
cluster context, each entry showing the object's kind icon and name. Activating an entry SHALL
open or focus that object's detail panel, in its own cluster context, following the same
open-or-focus convention as `resource-links`.

#### Scenario: Activating a favorite opens its panel
- **WHEN** the Favorites panel lists a pinned Pod and the user activates that entry
- **THEN** that Pod's detail panel opens, or is focused if already open, in its cluster context

#### Scenario: Grouped by cluster context
- **WHEN** pins exist across two cluster contexts
- **THEN** the Favorites panel groups entries under each context's name

#### Scenario: An empty favorites list
- **WHEN** no objects are pinned
- **THEN** the Favorites panel says there are no pinned objects, rather than showing nothing

### Requirement: A pinned object surviving its own absence
If a pinned object no longer exists in its cluster, the Favorites panel SHALL still list it
(pins are not removed automatically), and activating it SHALL open that object's panel showing
that it doesn't exist, matching `object-detail`'s existing behavior for a missing object.

#### Scenario: Pinned object was deleted
- **WHEN** a pinned Pod is deleted from the cluster
- **THEN** it remains listed in the Favorites panel, and activating it opens a panel reporting
  that the Pod no longer exists

### Requirement: Favorites is fully keyboard-operable
Pinning, unpinning, opening the Favorites panel, and activating an entry within it SHALL each be
reachable from the keyboard as well as the mouse, following `.claude/rules/keyboard-first.md`:
each is a registered command with a visible key hint.

#### Scenario: Pin by keystroke
- **WHEN** an object's detail panel has focus
- **THEN** the pin/unpin action is reachable by a keybinding shown in the panel's shortcut hints

#### Scenario: Navigating the Favorites panel by keyboard
- **WHEN** the Favorites panel has focus
- **THEN** the user can move between entries and activate the highlighted one without a mouse
