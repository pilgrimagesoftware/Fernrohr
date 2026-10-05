# Spec Delta

## Purpose

Lets the user act on a selected Kubernetes resource - delete it, force-kill it, edit its YAML, or
shell into a Pod's container - from the keyboard, not just inspect it.

## ADDED Requirements

### Requirement: Delete a resource with confirmation

The application SHALL provide a "delete" command that asks the user to confirm before deleting a
resource from the cluster. It SHALL be offered on the selected row of every resource list, Pods
included, and in the resource's detail panel - the object detail panel and the pod detail panel -
for every kind the API server's discovery lists the `delete` verb for, and for no other kind. The
command SHALL not act while focus is in a text field.

#### Scenario: Confirm deletes the resource

- **WHEN** the user invokes delete on a selected Pod and confirms the prompt
- **THEN** the Pod is deleted from the cluster and its row is removed once the delete is observed

#### Scenario: Delete a Secret from the Secrets list

- **WHEN** the user selects a Secret in the Secrets list, invokes delete, and confirms with Enter
- **THEN** the Secret is deleted from the cluster and its row is removed once the delete is observed

#### Scenario: Cancel leaves the resource untouched

- **WHEN** the user invokes delete on a selected resource and cancels the prompt
- **THEN** no delete request is sent and the row remains unchanged

#### Scenario: Delete request fails

- **WHEN** the cluster rejects the delete request, for example due to insufficient permissions
- **THEN** the application shows the failure reason and the row remains

#### Scenario: A kind that can't be deleted offers no delete

- **WHEN** discovery does not list the `delete` verb for a kind
- **THEN** neither its list nor its detail panel offers delete, by key or in the palette

#### Scenario: A detail panel shows the deleted resource is gone

- **WHEN** the user deletes the resource a detail panel shows, from that panel or elsewhere
- **THEN** the panel shows the resource deleted rather than its last state as if current, and no longer offers delete

### Requirement: Confirmations name objects distinctly and show their keys

Every confirmation dialog that names objects SHALL set each name apart from the surrounding text:
quoted, in the monospace code font, and in the theme's accent colour. Every confirmation dialog's
buttons SHALL keep their text labels and show the key bound to them in the live keymap - the
confirm button its Enter key, Cancel its Escape key - and pressing that key SHALL do what the
button does.

#### Scenario: The object's name stands out

- **WHEN** a delete confirmation asks about a Secret named `db-password`
- **THEN** the dialog's text shows “db-password” as its own run in the code font and accent colour

#### Scenario: Buttons show and honour their keys

- **WHEN** a confirmation dialog is open
- **THEN** its confirm button shows its Enter key and Cancel its Escape key, Enter confirms, and Escape cancels

### Requirement: Edit a resource's YAML

The application SHALL provide an "edit" command that opens a selected resource's manifest as
editable YAML and, on save, applies the edited manifest to the cluster. Edit SHALL be available for
every kind the API server's discovery lists the `patch` verb for, except Secret, whose values the
application never shows. The user SHALL be able to start it from a list panel's selected row (the
Pods panel and every other list), from a pod's detail panel, and from an object's detail panel, in
its fields or YAML view. Every route SHALL open the same edit view over that object's manifest.

#### Scenario: Edit from a list row

- **WHEN** a list panel (the Pods panel or any other list) has a row selected and the user runs edit
- **THEN** the object's manifest opens in the edit view

#### Scenario: Edit from a detail panel's YAML view

- **WHEN** a pod's or another object's detail panel is showing its YAML and the user runs edit
- **THEN** that object's manifest opens in the edit view

#### Scenario: A Secret is not editable

- **WHEN** the user runs edit on a Secret
- **THEN** no edit view opens, and the application says a Secret's values are hidden in this view

#### Scenario: A read-only kind offers no edit

- **WHEN** discovery does not list the `patch` verb for a kind, such as an aggregated metrics kind
- **THEN** neither its list nor its detail panel offers edit, and no Save button appears

#### Scenario: Edit and save applies the change

- **WHEN** the user edits a resource's YAML in the edit view and saves
- **THEN** the application sends the edited manifest to the cluster as an update

#### Scenario: Invalid YAML blocks save

- **WHEN** the user's edited content is not valid YAML or not a valid manifest for that resource kind
- **THEN** save is rejected with a reason and the cluster is not updated

#### Scenario: Server rejects the update

- **WHEN** the cluster rejects the update, for example a conflicting resourceVersion
- **THEN** the application shows the failure reason and the edit view keeps the user's content so they can retry

### Requirement: Shell into a Pod's container

The application SHALL provide a "shell" command, available only for a Pod with at least one running
container, that opens an interactive exec session into that container in a panel. It SHALL be
offered on the selected Pods row and in the pod's detail panel.

#### Scenario: Open a shell into a running pod

- **WHEN** the user invokes shell on a selected running Pod with one container
- **THEN** an interactive exec session to that container opens in a panel and accepts keyboard input

#### Scenario: Shell from a pod's detail panel

- **WHEN** the user invokes shell in the detail panel of a Pod with one running container
- **THEN** an interactive exec session to that container opens, as it would from the Pods row

#### Scenario: Shell unavailable for a pod with no running container

- **WHEN** the user invokes shell on a Pod with no running container
- **THEN** the command has no effect and is not offered in the palette for that Pod

#### Scenario: Multi-container pod prompts for a container

- **WHEN** the user invokes shell on a Pod with more than one container
- **THEN** the application asks which container to shell into before opening the session

#### Scenario: Exec session ends when the container stops

- **WHEN** the container backing an open shell session exits or is deleted
- **THEN** the session panel indicates the session has ended and retains its transcript
