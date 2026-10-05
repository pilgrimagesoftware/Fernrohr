# Spec Delta

## Purpose

Lets the user act on a selected Kubernetes resource - delete it, force-kill it, edit its YAML, or
shell into a Pod's container - from the keyboard, not just inspect it.

## ADDED Requirements

### Requirement: Delete a resource with confirmation

The application SHALL provide a "delete" command on a resource-browser row that asks the user to
confirm before deleting that resource from the cluster.

#### Scenario: Confirm deletes the resource

- **WHEN** the user invokes delete on a selected Pod and confirms the prompt
- **THEN** the Pod is deleted from the cluster and its row is removed once the delete is observed

#### Scenario: Cancel leaves the resource untouched

- **WHEN** the user invokes delete on a selected resource and cancels the prompt
- **THEN** no delete request is sent and the row remains unchanged

#### Scenario: Delete request fails

- **WHEN** the cluster rejects the delete request, for example due to insufficient permissions
- **THEN** the application shows the failure reason and the row remains

### Requirement: Force-kill a resource without confirmation

The application SHALL provide a "force kill" command on a resource-browser row that deletes that
resource immediately with zero grace period and without a confirmation prompt.

#### Scenario: Force kill deletes immediately

- **WHEN** the user invokes force kill on a selected Pod
- **THEN** a delete request with zero grace period is sent immediately with no confirmation step

### Requirement: Edit a resource's YAML

The application SHALL provide an "edit" command that opens a selected resource's manifest as
editable YAML and, on save, applies the edited manifest to the cluster. Edit SHALL be available for
every kind except Secret, whose values the application never shows. The user SHALL be able to start
it from a list panel's selected row (the Pods panel and every other list), from a pod's detail panel,
and from an object's detail panel, in its fields or YAML view. Every route SHALL open the same edit
view over that object's manifest.

#### Scenario: Edit from a list row

- **WHEN** a list panel (the Pods panel or any other list) has a row selected and the user runs edit
- **THEN** the object's manifest opens in the edit view

#### Scenario: Edit from a detail panel's YAML view

- **WHEN** a pod's or another object's detail panel is showing its YAML and the user runs edit
- **THEN** that object's manifest opens in the edit view

#### Scenario: A Secret is not editable

- **WHEN** the user runs edit on a Secret
- **THEN** no edit view opens, and the application says a Secret's values are hidden in this view

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
container, that opens an interactive exec session into that container in a panel.

#### Scenario: Open a shell into a running pod

- **WHEN** the user invokes shell on a selected running Pod with one container
- **THEN** an interactive exec session to that container opens in a panel and accepts keyboard input

#### Scenario: Shell unavailable for a pod with no running container

- **WHEN** the user invokes shell on a Pod with no running container
- **THEN** the command has no effect and is not offered in the palette for that Pod

#### Scenario: Multi-container pod prompts for a container

- **WHEN** the user invokes shell on a Pod with more than one container
- **THEN** the application asks which container to shell into before opening the session

#### Scenario: Exec session ends when the container stops

- **WHEN** the container backing an open shell session exits or is deleted
- **THEN** the session panel indicates the session has ended and retains its transcript
