# resource-links Specification

## Purpose
How a detail view represents a reference to another Kubernetes object, and what following that
reference does: open or focus the referenced object's panel, in the right cluster context.

## Requirements

### Requirement: References to viewable objects are links
A detail view SHALL show every reference to another object whose kind the application can
display as a link, and SHALL show a reference to a kind it can't display as plain text, not
styled as a link.

#### Scenario: Reference to a viewable kind
- **WHEN** a detail view shows a field naming an object whose kind the application has a panel for
- **THEN** that name is shown as a link

#### Scenario: Reference to a kind with no viewer
- **WHEN** a detail view shows a field naming an object whose kind the application has no panel for
- **THEN** that name is shown as plain text, not styled as a link, and activating it does nothing

#### Scenario: A kind gains a viewer
- **WHEN** the application gains a panel for a kind that detail views already reference
- **THEN** every existing reference to that kind is shown as a link, with no change to the views
  that show the references

### Requirement: Following a link opens the referenced object's panel
Following a link SHALL open a panel for the referenced object in the same cluster context as the
panel the link was followed from, or SHALL focus that object's panel if one is already open
there.

#### Scenario: Following a link to an object with no open panel
- **WHEN** the user follows a link to an object that has no open panel in that cluster context
- **THEN** a panel for that object opens and takes focus, and the panel the link was followed
  from stays open

#### Scenario: Following a link to an already-open object
- **WHEN** the user follows a link to an object whose panel is already open in that cluster
  context
- **THEN** that existing panel is focused and no second panel opens

#### Scenario: Same-named objects in different clusters
- **WHEN** two panels from different cluster contexts each link to an object with the same kind,
  namespace and name
- **THEN** each link opens or focuses the panel for its own cluster context's object

#### Scenario: Namespaced reference resolves in the referencing object's namespace
- **WHEN** a namespaced object references another namespaced object by name alone
- **THEN** the link opens that name in the referencing object's namespace

### Requirement: Links are reachable by keyboard
Every link in a detail view SHALL be followable without a mouse, following the application's
visible-shortcut-hint convention.

#### Scenario: Following a link by keystroke
- **WHEN** a detail view with at least one link has focus and the user presses the view's
  follow-link key
- **THEN** the user can choose any of that view's links and follow it without using the mouse

#### Scenario: Choosing a link doesn't follow the pointer
- **WHEN** the link chooser is open and the pointer moves over an entry without clicking
- **THEN** the chosen entry doesn't change

#### Scenario: Cancelling the link chooser
- **WHEN** the link chooser is open and the user presses Escape
- **THEN** the chooser closes without following a link, and focus returns to the detail view

#### Scenario: The follow-link key is shown
- **WHEN** a detail view with at least one link has focus
- **THEN** the view shows its follow-link key, as currently bound, in its shortcut hints

### Requirement: A followed link survives its target's absence
Following a link to an object that doesn't exist (deleted, or never created, like a Secret
mounted as `optional`) SHALL open that object's panel showing that the object doesn't exist,
not an error or nothing.

#### Scenario: Link to a deleted object
- **WHEN** the user follows a link to an object that has since been deleted
- **THEN** the object's panel opens and shows that the object no longer exists
