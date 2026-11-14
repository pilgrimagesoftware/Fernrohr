# Spec Delta

## Purpose

Let the user roll a Helm release back to an earlier revision or uninstall it from Fernrohr, with
Helm's own semantics, behind confirmations, against the same cluster endpoint Fernrohr's connection
uses.

## ADDED Requirements

### Requirement: Actions run through the user's helm CLI

Rollback and Uninstall SHALL be performed by the user's `helm` binary, version 3 or 4,, found at a path set
in Settings or else on the login shell's PATH, so Helm's hooks, resource updates and history behave
as they do from a terminal.

#### Scenario: helm at a Settings path

- **WHEN** Settings names a `helm` 3 or 4 binary that isn't on the PATH
- **THEN** Rollback and Uninstall are available and run that binary

### Requirement: helm reaches the cluster the way the connection does

The application SHALL point `helm` at the connection's effective endpoint, including a bound
tunnel, with the context's own credentials, and SHALL NOT change the user's kubeconfig or
environment.

#### Scenario: A tunneled context

- **WHEN** the context connects through an SSH or command tunnel and the user rolls back a release
- **THEN** `helm` reaches the cluster through that same tunnel, and the rollback succeeds without the
  user running anything else

### Requirement: Unavailable without helm

Without a usable `helm` 3 or 4, Rollback and Uninstall SHALL be shown disabled with the reason, in their
menu items, buttons and the palette.

#### Scenario: helm is missing

- **WHEN** no `helm` binary can be found
- **THEN** Rollback and Uninstall show as disabled, and their tooltip or palette entry says `helm`
  3 wasn't found and where to set its path

### Requirement: Rollback to an earlier revision

A release detail panel SHALL offer Rollback, from its History tab and as a command, that rolls the
release back to the revision selected in History (by default the one before the current
revision). It SHALL first show a recoverable confirmation naming the release, its namespace, the
current revision and the target revision with its chart and app version. Enter MAY confirm it and
Escape SHALL cancel it.

#### Scenario: Roll back to revision 3

- **WHEN** release `web` is at revision 5, the user selects revision 3 in History and runs Rollback,
  then confirms
- **THEN** `helm` rolls `web` back to revision 3, and the panel shows a new revision 6 whose
  description says it is a rollback to 3

#### Scenario: Cancelled

- **WHEN** the user presses Escape in the Rollback confirmation
- **THEN** nothing is run and the release is unchanged

### Requirement: Uninstall

A release's row and its detail panel SHALL offer Uninstall. It SHALL first show an irreversible
confirmation that names the release and its namespace and lists how many objects the deployed
manifest declares. The dialog SHALL open with focus on Cancel. Enter SHALL NOT confirm it; the
destructive button runs only when clicked, reached with Tab and activated, or triggered by its
deliberate shortcut, shown as a key.

#### Scenario: Uninstall from the list

- **WHEN** the user selects release `web` in the list, runs Uninstall, and confirms with the
  deliberate shortcut
- **THEN** `helm` uninstalls `web`, and its row disappears once its revision Secrets are gone

#### Scenario: Enter cancels

- **WHEN** the Uninstall confirmation is open and the user presses Enter
- **THEN** the dialog closes and nothing is run

### Requirement: Progress and failure

While an action runs the panel SHALL show that it is running and SHALL NOT start a second action on
the same release. A failure SHALL be shown on the panel with `helm`'s own error message, not as a
silent no-op. Credentials SHALL NOT appear in what is shown or logged.

#### Scenario: helm refuses

- **WHEN** `helm` exits non-zero because a pre-delete hook failed
- **THEN** the panel shows that Uninstall failed with `helm`'s message, and the release is still
  listed

### Requirement: Keyboard routes

Rollback and Uninstall SHALL be registered commands with default bindings, overridable in
`keymap.toml`, listed in the command palette while a release is selected or open, offered in the
release row's context menu, and shown in the panels' hint rows.

#### Scenario: Rollback without the mouse

- **WHEN** the user opens History by its key, moves to revision 3 with the arrow keys, presses
  Rollback's key and then Enter
- **THEN** the release is rolled back to revision 3
