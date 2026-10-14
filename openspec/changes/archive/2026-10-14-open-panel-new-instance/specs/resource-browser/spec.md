# Spec Delta

## ADDED Requirements

### Requirement: Opening a list row in a new panel
Every list panel whose rows open a detail panel - the object lists and the Pods list - SHALL offer
an Open in New Panel route for a row. It opens that row's detail panel as a new instance with
keyboard focus, even when the row's panel is already open. From the keyboard it SHALL be a command
for the list with a default key, in the palette while the list has focus. From the mouse it SHALL
be an `alt`-click on the row and an Open in New Panel item in the row's context menu.

#### Scenario: From the keyboard
- **WHEN** a Services list has focus with a Service selected whose detail panel is already open,
  and the user presses `shift-enter`
- **THEN** a second detail panel for that Service opens with keyboard focus

#### Scenario: Alt-click
- **WHEN** the user `alt`-clicks a Deployments row whose detail panel is already open
- **THEN** a second detail panel for that Deployment opens

#### Scenario: Context menu
- **WHEN** the user right-clicks a row of an object list and chooses Open in New Panel
- **THEN** a new detail panel for that row's object opens

#### Scenario: Typing is not intercepted
- **WHEN** a list panel's filter field has focus and the user presses `shift-enter`
- **THEN** no panel opens

## MODIFIED Requirements

### Requirement: Pod rows have a context menu
Right-clicking a pod row SHALL select it and open a context menu offering Quick Look, Open Details,
Open in New Panel, Logs and YAML, each running the same command as its key.

#### Scenario: Logs from the context menu
- **WHEN** the user right-clicks a pod row and chooses Logs
- **THEN** that pod's logs open, as pressing `l` on the row would

#### Scenario: Open in New Panel from the context menu
- **WHEN** the user right-clicks a pod row whose detail panel is already open and chooses Open in
  New Panel
- **THEN** a second detail panel for that pod opens, as pressing `shift-enter` on the row would
