# Spec Delta

## MODIFIED Requirements

### Requirement: Filtering the Resource panel

The Resource panel SHALL provide a filter box pinned to its bottom edge that narrows the visible
rows across every section at once. Matching SHALL be a case-insensitive substring test against a
row's label, kind, resource plural and API group.

#### Scenario: A filter narrows every section at once

- **WHEN** the user types a substring that matches kinds in more than one
  section
- **THEN** only matching rows are shown, in each of those sections

#### Scenario: Matches are visible even in a collapsed section

- **WHEN** a section is collapsed and the filter matches a row inside it
- **THEN** that section is shown expanded with the matching row visible

#### Scenario: Clearing the filter restores the previous shape

- **WHEN** the user clears the filter
- **THEN** every section returns to the collapse state it had before the filter
  was typed

#### Scenario: A filter matching nothing says so

- **WHEN** the filter matches no kind in the cluster
- **THEN** the panel shows no sections and reports that nothing matched

#### Scenario: Group names are searchable

- **WHEN** the user filters on an API group name such as `batch`
- **THEN** kinds in that group match, even where the group name is in neither
  the row's label nor its kind

### Requirement: Every discovered kind opens a list panel

Opening any kind from the Resource panel SHALL open a live list panel of that kind's objects for
its cluster context, and SHALL NOT open a placeholder. Every list panel SHALL provide the same
behavior this capability requires of the Pods table: live updates, namespace scoping, filtering
and sorting, resizable and reorderable columns, a plural title, a shared reference-counted watch,
and full keyboard operation.

#### Scenario: Opening Deployments from the Resource panel

- **WHEN** the user opens Deployments from the Resource panel
- **THEN** a panel titled "Deployments" lists the cluster's Deployments and updates live as they
  are created, changed and deleted

#### Scenario: A cluster-scoped kind

- **WHEN** the user opens Nodes from the Resource panel
- **THEN** the list shows every Node, with no namespace selector and no Namespace column

#### Scenario: A custom resource kind

- **WHEN** the user opens a CRD-defined kind from the Resource panel
- **THEN** a list panel opens showing that kind's objects with the base columns

#### Scenario: Two lists of one kind share a watch

- **WHEN** two Services panels are open against the same cluster
- **THEN** only one Service watch stream exists for that cluster, and it stops when the last of
  those panels closes

#### Scenario: A kind the user cannot list

- **WHEN** the user opens a kind they are not permitted to list
- **THEN** the panel shows the API server's refusal in place of rows rather than an empty table

#### Scenario: Restored with the window

- **WHEN** the application restarts with a Services list panel in a saved window layout
- **THEN** the panel reopens as a Services list in the same cluster context, with its namespace
  selection

### Requirement: List columns follow the kind

Every list panel SHALL show Name and Age, and Namespace for a namespaced kind. A kind with no
kind-specific columns SHALL show only these base columns. Pods keep the columns their existing
requirement defines.

#### Scenario: A Services list shows service columns

- **WHEN** a Services list panel is open
- **THEN** each row shows the Service's name, namespace, type, cluster IP, external IP, ports and
  age

#### Scenario: Per-kind columns sort and resize like any other

- **WHEN** the user clicks the Available header in a Deployments list
- **THEN** rows sort by available replica count, numerically rather than as text

#### Scenario: A kind without extra columns

- **WHEN** a Roles list panel is open
- **THEN** each row shows only name, namespace and age

### Requirement: The namespace scope indicator names a matching set

A namespaced panel's namespace scope indicator SHALL show a saved namespace set's name when the
panel's scope is an include set whose namespaces are exactly that set's namespaces. Otherwise the
indicator SHALL read as before, as a count of included namespaces or as "all namespaces".

#### Scenario: A switched panel names its set

- **WHEN** the set `team-workloads` contains `team-a` and `team-b`, and a panel is switched to it
- **THEN** the panel's namespace indicator reads `team-workloads`

#### Scenario: A hand-built scope reads as a count

- **WHEN** the user builds an include set of `team-a` and `team-b` by hand and no saved set has
  exactly those namespaces
- **THEN** the panel's namespace indicator reads as a count, not a set name

#### Scenario: An edited set no longer matches

- **WHEN** a panel is switched to `team-workloads` containing `team-a`, and the user then adds
  `team-c` to that set
- **THEN** the panel's namespace indicator reads as a count, while the set itself shows three
  namespaces

### Requirement: Quick look at a selected pod

With a pod selected in a Pods table, the Quick Look action - Space, a "Pods: Quick Look" command, or
the row's context menu - SHALL open a popover anchored to the selected row showing the pod's name
and namespace, phase (colored, with its text), ready containers, restarts, age, node, pod IP, owner
as a link, each container's image and state, and its most recent Warning event if it has one.

#### Scenario: Opening a quick look
- **WHEN** a Pods table has focus with a crashing pod selected and the user presses Space
- **THEN** a popover beside that row shows the pod's phase, `1/2` ready, its restart count, each
  container's image and state, and the latest `BackOff` warning

#### Scenario: Scanning several pods
- **WHEN** a quick look is open and the user presses Down
- **THEN** the table's selection moves to the next pod and the popover shows that pod, without
  closing and reopening

#### Scenario: Going to the full detail
- **WHEN** a quick look is open and the user presses Enter
- **THEN** the pod's detail panel opens (or is focused if already open) and the popover closes

#### Scenario: Live while open
- **WHEN** a quick look is open and the pod's ready count changes
- **THEN** the popover updates in place

#### Scenario: The pod goes away
- **WHEN** the pod is deleted while its quick look is open
- **THEN** the popover says the pod no longer exists rather than closing silently

### Requirement: Filtering the namespace picker

A namespaced panel's namespace picker SHALL show a filter input when it opens, with keyboard focus
in it. Typing SHALL narrow the listed namespaces to those whose name contains the typed text,
ignoring case. The "All namespaces" entry SHALL remain listed first regardless of the filter.

#### Scenario: Typing narrows the list

- **WHEN** the cluster has namespaces `default`, `kube-system`, `kube-public` and `payments`, and
  the user opens the picker and types `KUBE`
- **THEN** the picker lists "All namespaces", `kube-system` and `kube-public`, and not `default`
  or `payments`

#### Scenario: Picking several matches in a row

- **WHEN** the filter is `kube` and the user toggles `kube-system`, then `kube-public`
- **THEN** the picker stays open with the filter still `kube`
- **AND** the panel is scoped to both namespaces, with the button reading `2 namespaces`

#### Scenario: Keyboard toggle

- **WHEN** the picker is open, the filter is `pay`, and the user presses Down to highlight
  `payments`, then Enter
- **THEN** `payments` is toggled in the panel's scope

#### Scenario: Escape clears, then closes

- **WHEN** the filter holds `kube` and the user presses Escape
- **THEN** the filter is empty and every namespace is listed again
- **AND** pressing Escape again closes the picker

#### Scenario: No match

- **WHEN** the user types `zzz` and no namespace contains it
- **THEN** the picker lists "All namespaces" and a "No matching namespaces" message

#### Scenario: Filter resets on reopen

- **WHEN** the user types `kube`, closes the picker, and opens it again
- **THEN** the filter is empty and every namespace is listed

### Requirement: Opening a list row in the background

Every list panel SHALL offer an "Open in Background" command that opens the selected row's
detail panel the way activating the row does, except that the new panel is added as an inactive
tab and keyboard focus and the list's selection stay where they were.

#### Scenario: Background open from the keyboard

- **WHEN** a Pods panel has focus with a row selected and the user presses `cmd-enter`
- **THEN** that pod's detail panel opens as an inactive tab, and the Pods panel keeps keyboard focus and the same selected row

#### Scenario: Background open with the mouse

- **WHEN** the user `cmd`-clicks the third row of a Deployments list whose first row is selected
- **THEN** the third row's Deployment opens as an inactive tab, and the first row stays selected with focus in the list

#### Scenario: Middle-click

- **WHEN** the user middle-clicks a row in any list panel
- **THEN** that row's object opens in the background, as with the modifier-click

#### Scenario: The list stays visible in its own group

- **WHEN** a background open places the new panel in the same tab group as the list
- **THEN** the list remains the active tab of that group

#### Scenario: Already open

- **WHEN** the user opens in the background an object whose panel is already open
- **THEN** no panel opens, no tab is activated, and focus stays in the list

#### Scenario: Typing is not intercepted

- **WHEN** a list panel's filter field has focus and the user presses `cmd-enter`
- **THEN** no panel opens

## ADDED Requirements

### Requirement: A Resource panel filter shows matching sections expanded

While a Resource panel filter is active, a section holding at least one matching row SHALL be
rendered expanded regardless of its collapse state, and a section holding no matching row SHALL be
hidden.

#### Scenario: A match inside a collapsed section

- **WHEN** a section is collapsed and the filter matches a row inside it
- **THEN** that section is shown expanded with the matching row visible

### Requirement: A Resource panel filter keeps the collapse state

Filtering the Resource panel SHALL NOT change its sections' stored collapse state, so clearing
the filter restores the panel to the shape the user left it in.

#### Scenario: Clearing the filter

- **WHEN** the user clears the filter
- **THEN** every section returns to the collapse state it had before the filter was typed

### Requirement: A cluster-scoped kind's list has no namespace scoping

A list panel of a cluster-scoped kind SHALL have no namespace selector and no Namespace
column.

#### Scenario: Nodes

- **WHEN** the user opens Nodes from the Resource panel
- **THEN** the list has no namespace selector and no Namespace column

### Requirement: Workload lists show kubectl's default columns

A list of one of these workload kinds SHALL also show that kind's columns, matching what
`kubectl get` shows by default: Deployment - Ready (ready/desired), Up-to-date, Available;
ReplicaSet - Desired, Current, Ready; StatefulSet - Ready (ready/desired); DaemonSet - Desired,
Current, Ready, Up-to-date, Available; Job - Status, Completions, Duration; CronJob - Schedule,
Suspend, Active, Last schedule.

#### Scenario: A Deployments list

- **WHEN** a Deployments list panel is open
- **THEN** each row also shows Ready as ready/desired, Up-to-date and Available

### Requirement: Configuration and network lists show kubectl's default columns

A list of one of these kinds SHALL also show that kind's columns, matching what `kubectl get`
shows by default: ConfigMap - Data (key count); Secret - Type, Data (key count); Service - Type,
Cluster IP, External IP, Ports; Ingress - Class, Hosts, Address, Ports; Endpoints - Endpoints;
EndpointSlice - Address type, Ports, Endpoints; NetworkPolicy - Pod selector.

#### Scenario: A Services list

- **WHEN** a Services list panel is open
- **THEN** each row shows the Service's name, namespace, type, cluster IP, external IP, ports and age

### Requirement: Storage lists show kubectl's default columns

A list of one of these storage kinds SHALL also show that kind's columns, matching what
`kubectl get` shows by default: PersistentVolumeClaim - Status, Volume, Capacity, Access modes,
Storage class; PersistentVolume - Capacity, Access modes, Reclaim policy, Status, Claim, Storage
class; StorageClass - Provisioner, Reclaim policy, Volume binding mode.

#### Scenario: A PersistentVolumeClaims list

- **WHEN** a PersistentVolumeClaims list panel is open
- **THEN** each row also shows the claim's status, volume, capacity, access modes and storage class

### Requirement: Cluster and access lists show kubectl's default columns

A list of one of these kinds SHALL also show that kind's columns, matching what `kubectl get`
shows by default: Node - Status, Roles, Version, Internal IP; Namespace - Status; ServiceAccount -
Secrets (count); RoleBinding and ClusterRoleBinding - Role, Subjects.

#### Scenario: A Nodes list

- **WHEN** a Nodes list panel is open
- **THEN** each row also shows the Node's status, roles, version and internal IP

### Requirement: A panel switched to a namespace set keeps its namespaces

Because a panel stores the namespaces it was switched to rather than a reference to the set,
deleting or editing a namespace set SHALL leave an already-switched panel's scope and indicator
unchanged until the panel is switched to a set again.

#### Scenario: The set is edited after switching

- **WHEN** a panel is switched to `team-workloads` containing `team-a` and `team-b`, and the user then adds `team-c` to that set
- **THEN** the panel's scope is still `team-a` and `team-b`

### Requirement: A quick look opens the pod's details

The quick look popover SHALL offer an Open Details action, by button and by Enter, that opens the
pod's detail panel and closes the popover.

#### Scenario: Enter opens the details

- **WHEN** a quick look is open and the user presses Enter
- **THEN** the pod's detail panel opens and the popover closes

### Requirement: A quick look closes back to the table

Space or Escape SHALL close the quick look popover and return keyboard focus to the Pods
table.

#### Scenario: Escape closes it

- **WHEN** a quick look is open and the user presses Escape
- **THEN** the popover closes and the Pods table has keyboard focus

### Requirement: Toggling keeps the namespace picker open

Toggling a namespace in the namespace picker SHALL keep the picker open with its filter text, so
several namespaces can be picked in a row.

#### Scenario: Two toggles in a row

- **WHEN** the filter is `kube` and the user toggles `kube-system`, then `kube-public`
- **THEN** the picker stays open with the filter still `kube`

### Requirement: The namespace picker works from the keyboard

In the namespace picker, Up and Down SHALL move through the filtered entries, and Enter SHALL
toggle the highlighted one. Escape SHALL clear non-empty filter text, and close the picker when
the text is already empty.

#### Scenario: Escape clears, then closes

- **WHEN** the filter holds `kube` and the user presses Escape twice
- **THEN** the first Escape empties the filter and the second closes the picker

### Requirement: The namespace picker says when nothing matches

When the namespace picker's filter matches no namespace, the picker SHALL say so.

#### Scenario: No match

- **WHEN** the user types `zzz` and no namespace contains it
- **THEN** the picker shows a "No matching namespaces" message

### Requirement: The namespace picker's filter resets on close

The namespace picker's filter text SHALL NOT survive closing the picker.

#### Scenario: Reopening

- **WHEN** the user types `kube`, closes the picker, and opens it again
- **THEN** the filter is empty and every namespace is listed

### Requirement: Open in Background has a key and is discoverable

The Open in Background command SHALL be bound by default to the platform modifier with Enter
(`cmd-enter` on macOS, `ctrl-enter` elsewhere), SHALL NOT fire while a text field has focus, and
SHALL appear in the command palette and the panel's hint row.

#### Scenario: Typing is not intercepted

- **WHEN** a list panel's filter field has focus and the user presses `cmd-enter`
- **THEN** no panel opens

### Requirement: A modifier-click or middle-click opens a row in the background

A click on a list row with the platform modifier held, or a middle-click on a row, SHALL open
that row's object in the background as Open in Background does, without changing the
selection.

#### Scenario: Middle-click

- **WHEN** the user middle-clicks a row in any list panel
- **THEN** that row's object opens as an inactive tab and the selection is unchanged

### Requirement: Opening an already-open object in the background changes nothing

If an object's panel is already open, Open in Background SHALL change nothing: it neither
activates that tab nor moves focus.

#### Scenario: Already open

- **WHEN** the user opens in the background an object whose panel is already open
- **THEN** no panel opens, no tab is activated, and focus stays in the list
