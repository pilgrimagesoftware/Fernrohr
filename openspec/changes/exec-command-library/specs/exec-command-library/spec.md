## ADDED Requirements

### Requirement: Exec library entries
Fernrohr SHALL keep a library of exec entries, each with a name unique ignoring case and surrounding whitespace, a non-empty command, an optional container, and a target. The command SHALL be split into arguments with POSIX shell quoting and run without a shell.

#### Scenario: Duplicate name
- **WHEN** the user saves an entry named " stream INFO " while "Stream info" exists
- **THEN** the dialog shows an error that the name is taken, and nothing is saved

#### Scenario: Unbalanced quote
- **WHEN** the entry's command is `echo "unterminated`
- **THEN** the dialog shows a command error, and Save and Save and Run are refused

### Requirement: Entry targets
An entry's target SHALL be one of three kinds. It is any pod; a pod selector, made of an optional namespace and an optional label selector in `kubectl -l` syntax; or a workload, made of a Deployment, StatefulSet, DaemonSet or Service with a namespace and a name.

#### Scenario: Selector entry
- **WHEN** the user saves an entry named "Stream info" with command `nats stream info X`, namespace `<namespace>` and selector `app.kubernetes.io/component=nats-box`
- **THEN** the library lists "Stream info" with that command and scope

#### Scenario: Workload needs a name
- **WHEN** the user saves a workload entry with no workload name
- **THEN** the dialog shows an error on the name field, and nothing is saved

### Requirement: The library survives a restart
The library SHALL be stored in a hand-editable `exec-library.toml` in Fernrohr's preferences directory, written atomically on every change and kept in entry order. If the file cannot be parsed, Fernrohr SHALL leave it unchanged, show the library as empty and read-only, and report the file and the error.

#### Scenario: Restart
- **WHEN** the user saves two entries and restarts Fernrohr
- **THEN** the library lists both entries in the same order

#### Scenario: Broken file
- **WHEN** `exec-library.toml` contains invalid TOML at startup
- **THEN** the library shows the parse error, offers no edits, and the file is unchanged on disk

### Requirement: Running an entry opens a shell panel running its command
Running an entry SHALL open a shell panel that runs the entry's command in the resolved pod and container with a terminal attached, in place of the interactive shell. The panel follows the shell panel's existing behavior, and it SHALL show the entry name and pod in its title. A panel restored after a restart SHALL show the session as ended and SHALL NOT run the command again.

#### Scenario: Command finishes
- **WHEN** an entry whose command prints output and exits with status 0 is run
- **THEN** the panel shows the output and an ended notice with status 0, and the output can be selected and copied

#### Scenario: Restored panel
- **WHEN** Fernrohr restarts with a library command panel in the saved layout
- **THEN** the panel is restored as ended and no command is sent to the cluster

### Requirement: Pod resolution
Running an entry SHALL resolve the pods it targets as follows:
- an any-pod entry run from a pod uses that pod;
- a pod-selector entry uses the Running pods that match its selector, in its namespace or in all namespaces when none is set;
- a workload entry uses the Running pods matched by the workload's own selector.

#### Scenario: One matching pod
- **WHEN** a selector entry matches exactly one Running pod
- **THEN** its command runs in that pod without a chooser

#### Scenario: No matching pod
- **WHEN** a selector entry matches only Pending pods
- **THEN** an error names the entry and its namespace and selector, and no panel opens

### Requirement: Choosing among matching pods
With one resolved pod, the entry SHALL run there. With several, a chooser SHALL list them in namespace-then-name order with the first preselected. With none, Fernrohr SHALL report an error naming the entry and its scope and SHALL NOT open a panel.

#### Scenario: Several matching pods
- **WHEN** a workload entry for a Deployment with three Running pods is run and the user presses Enter in the chooser
- **THEN** the command runs in the first listed pod

### Requirement: Container selection
The container SHALL be the entry's container. Otherwise it is the pod's `kubectl.kubernetes.io/default-container` annotation, and otherwise the pod's first container. An entry naming a container the pod lacks SHALL report an error and open no panel.

#### Scenario: Default container annotation
- **WHEN** an entry with no container runs in a pod annotated `kubectl.kubernetes.io/default-container: app` whose first container is `sidecar`
- **THEN** the command runs in the `app` container

#### Scenario: Missing container
- **WHEN** an entry naming container `tools` runs in a pod with no `tools` container
- **THEN** an error names the container and the pod, and no panel opens

### Requirement: Exec Library window
Fernrohr SHALL provide an Exec Library window that lists the entries with a filter and runs, creates, edits, duplicates and deletes them, each through a registered command with a keybinding shown in the window. Run SHALL use the context of the cluster panel the window was opened from, and SHALL ask which connected context to use when there is none. Deleting an entry SHALL ask for confirmation with focus on Cancel.

#### Scenario: Run from the keyboard
- **WHEN** the user opens the Exec Library from the command palette while a Pods panel is focused, types part of an entry's name, and presses Enter
- **THEN** the entry runs in that panel's context

#### Scenario: Delete needs a deliberate confirmation
- **WHEN** the user triggers Delete on an entry and presses Enter in the confirmation
- **THEN** the confirmation closes and the entry is still in the library

### Requirement: Run from Library on a resource
Pod, Deployment, StatefulSet, DaemonSet and Service rows and detail panels SHALL offer Run from Library…, through their context menu and a command scoped to that row or panel. It lists only the entries that apply to the resource and runs the chosen one there.

#### Scenario: Only matching entries offered
- **WHEN** the user opens Run from Library… on a pod labelled `app=web` and the library holds an any-pod entry, a selector entry for `app=web`, and a selector entry for `app=db`
- **THEN** the list offers the any-pod and `app=web` entries only

### Requirement: Which entries apply to a resource
An entry SHALL apply to a Pod when it is an any-pod entry, when it is a selector entry whose namespace matches or is unset and whose selector matches the pod's labels, or when it is a workload entry whose cached workload selects the pod. An entry SHALL apply to a workload or Service when it targets that object or selects any of its pods.

#### Scenario: Workload entry on its pod
- **WHEN** the user opens Run from Library… on a pod of a cached Deployment that a workload entry targets
- **THEN** that entry is offered

### Requirement: Save to Library from a resource
Pod, Deployment, StatefulSet, DaemonSet and Service rows and detail panels SHALL offer Save to Library…, through their context menu and a scoped command. It opens the entry dialog prefilled from the resource, with every field editable. The dialog SHALL offer Save, Save and Run, and Cancel, each showing its key. Save and Run SHALL save the entry and then run it on the resource it was created from.

#### Scenario: Save and run from a Deployment
- **WHEN** the user opens Save to Library… on a Deployment, enters a name and command, and presses the Save and Run key
- **THEN** the entry is saved with that Deployment as its target and its command runs in one of the Deployment's Running pods

### Requirement: Prefill from a resource
Save to Library… on a Pod SHALL prefill a selector target with the pod's namespace and its controller's selector. When the controller is not cached, it SHALL use the pod's labels without per-revision hash labels. On a workload or Service it SHALL prefill a workload target naming that object.

#### Scenario: Owner selector
- **WHEN** the user opens Save to Library… on a pod whose ReplicaSet and Deployment are cached
- **THEN** the prefilled selector is the Deployment's selector

#### Scenario: Pod prefill drops hash labels
- **WHEN** the user opens Save to Library… on a pod whose labels include `pod-template-hash` and whose ReplicaSet is not cached
- **THEN** the prefilled selector has the pod's other labels and no `pod-template-hash`

### Requirement: The entry dialog is keyboard-operable
The entry dialog SHALL put focus on the first empty required field when it opens, move between every field and button with Tab and Shift-Tab, save with Enter, save and run with its own key, and cancel with Escape, returning focus to where it was. Validation errors SHALL appear next to their field.

#### Scenario: Create without the mouse
- **WHEN** the user runs Save to Library… from the palette on a selected pod, types a name, tabs to the command, types it, and presses Enter
- **THEN** the entry is saved and focus returns to the pod row
