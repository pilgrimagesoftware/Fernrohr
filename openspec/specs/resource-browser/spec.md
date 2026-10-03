## Purpose

A panel that shows a live, continuously updated table of a Kubernetes resource kind (Pods in this
change), scoped by namespace and refined by filtering and sorting, backed by a shared watch stream.

## Requirements

### Requirement: Live Pods table

A resource-browser panel SHALL display Pods for its connected cluster in a table and SHALL reflect
create, update, and delete events from the cluster without the user refreshing.

#### Scenario: Initial population

- **WHEN** a Pods panel opens against a connected cluster
- **THEN** the table lists the current Pods with at least name, namespace, ready count, status, restarts, and age

#### Scenario: Live create and delete

- **WHEN** a Pod is created in the cluster while the panel is open
- **THEN** a row for that Pod appears within a few seconds
- **AND** when that Pod is deleted, its row disappears

#### Scenario: Live status change

- **WHEN** an existing Pod's phase changes from Pending to Running
- **THEN** that row's status cell updates in place without reordering unrelated rows

### Requirement: Namespace scoping

A resource-browser panel SHALL let the user view a single namespace or all namespaces, SHALL persist the selection with the panel, and SHALL inherit the active context's default namespace when opened. A "warp all" action SHALL update every open namespaced panel in that context to the selected namespace and set that namespace as the context default.

#### Scenario: Warp all updates open panels

- **WHEN** the user invokes "Warp all to namespace" for a selected Pod in `team-a`
- **THEN** every open namespaced resource panel in the same context shows only `team-a`
- **AND** cluster-scoped panels are unchanged

#### Scenario: Restrict to one namespace

- **WHEN** the user selects namespace `kube-system`
- **THEN** the table shows only Pods in `kube-system`

#### Scenario: All namespaces

- **WHEN** the user selects "all namespaces"
- **THEN** the table shows Pods from every namespace the user can list

#### Scenario: Warp all affects future panels

- **WHEN** the context default is `team-a`
- **AND** the user opens another namespaced resource panel in that context
- **THEN** the new panel starts scoped to `team-a`

#### Scenario: Existing focused warp remains local

- **WHEN** the user invokes the existing focused-panel warp for `team-a`
- **THEN** only the focused panel changes namespace
- **AND** the context default and other panels are unchanged

### Requirement: Filtering and sorting

A resource-browser panel SHALL provide a text filter that narrows rows by substring match on the
resource name, and SHALL let the user sort by any displayed column, cycling ascending,
descending, and unsorted, with the current sort column and direction shown by an indicator on
that column's header.

#### Scenario: Text filter

- **WHEN** the user types `nginx` into the filter
- **THEN** only rows whose Pod name contains `nginx` remain visible
- **AND** clearing the filter restores all rows

#### Scenario: Clicking a header sorts ascending

- **WHEN** the user clicks an unsorted column's header
- **THEN** the table's rows sort by that column ascending, and that header shows an ascending
  indicator

#### Scenario: Clicking again reverses direction

- **WHEN** the user clicks an already-ascending-sorted column's header
- **THEN** the table's rows sort by that column descending, and that header shows a descending
  indicator

### Requirement: Columns are resizable and reorderable

A resource-browser table's columns SHALL be resizable by dragging a column boundary and
reorderable by dragging a column header to a new position.

#### Scenario: Resizing a column

- **WHEN** the user drags a column boundary in the Pods table
- **THEN** that column's width changes and neighboring columns adjust accordingly

#### Scenario: Reordering a column

- **WHEN** the user drags a column header to a new position
- **THEN** the table's column order reflects the new position, and each cell still shows the
  value for its own column, not the column now in that visual slot

### Requirement: List panel titles use the kind's plural form

A resource-browser panel showing a list of a kind SHALL title its tab and title bar with that
kind's plural display name, not its singular Kubernetes Kind name.

#### Scenario: Pods list panel is titled "Pods"

- **WHEN** a Pods list panel is open
- **THEN** its tab and title bar read "Pods", not "Pod"

### Requirement: Shared, reference-counted watches

The application SHALL maintain at most one watch stream per (cluster, resource kind) regardless of how
many panels display it, and SHALL stop that watch when the last panel using it closes.

#### Scenario: Two panels share one watch

- **WHEN** two Pods panels are open against the same cluster
- **THEN** only one Pod watch stream exists for that cluster

#### Scenario: Watch stops when unused

- **WHEN** the last panel displaying Pods for a cluster closes
- **THEN** the Pod watch stream for that cluster is torn down

#### Scenario: Watch reconnects after a drop

- **WHEN** an active Pod watch stream is interrupted by a network error
- **THEN** the application re-establishes the watch and the table converges to the cluster's current state

### Requirement: Resource kinds are grouped into collapsible sections

The Resource panel SHALL present every discovered kind under exactly one named
section, rather than as one flat list, and SHALL show no kind twice. No kind may
be dropped by grouping: the union of the sections is exactly the set of kinds the
cluster's API discovery reported.

A kind's section SHALL be determined by its API group **and** its resource
plural together. A section SHALL NOT be inferred from the API group alone.

#### Scenario: Grouping loses no kind

- **WHEN** a cluster's discovery reports a kind that matches no known section
- **THEN** that kind appears under Custom Resources
- **AND** the total count of rows across all sections equals the number of
  discovered kinds

#### Scenario: Opening a kind still works from a section

- **WHEN** the user opens a kind from within a section
- **THEN** the same panel opens as it would from a flat list

### Requirement: Resource kinds carry a category

Every discovered kind SHALL be assigned exactly one category from a fixed set -
Workloads, Config, Network, Storage, Cluster, Access Control, or Custom
Resources - and sections SHALL be presented in that fixed order, not
alphabetically, with Custom Resources last.

#### Scenario: Core-group kinds land in different sections

- **WHEN** a cluster reports `Pod`, `Service`, `ConfigMap`,
  `PersistentVolumeClaim`, `ServiceAccount` and `Namespace`, all of which are in
  the core API group
- **THEN** they appear under Workloads, Network, Config, Storage, Access
  Control and Cluster respectively

#### Scenario: A CRD lands in Custom Resources

- **WHEN** a cluster defines a CRD that is not one of the built-in Kubernetes
  kinds
- **THEN** it appears under Custom Resources

#### Scenario: Sections appear in a fixed order

- **WHEN** several sections are non-empty
- **THEN** they are ordered Workloads, Config, Network, Storage, Cluster, Access
  Control, Custom Resources

#### Scenario: An empty section takes no space

- **WHEN** a section holds no kinds for this cluster
- **THEN** it is not rendered at all

### Requirement: Sections are collapsible

Each rendered section SHALL have a header that collapses and expands its rows,
and SHALL show how many kinds it holds. Collapse state SHALL be held per window
and SHALL NOT be written to the user's preference file, so a new window opens
with every section expanded.

#### Scenario: Collapsing a section hides its rows

- **WHEN** the user collapses a section
- **THEN** that section's rows are hidden
- **AND** its header remains, still showing the section's name and kind count

#### Scenario: A new window is not collapsed

- **WHEN** a user collapsed sections in one window and then opens a new window
- **THEN** the new window's sections are all expanded

### Requirement: Filtering the Resource panel

The Resource panel SHALL provide a filter box pinned to its bottom edge that
narrows the visible rows across every section at once. Matching SHALL be a
case-insensitive substring test against a row's label, kind, resource plural and
API group.

While a filter is active, a section holding at least one matching row SHALL be
rendered expanded regardless of its collapse state, and a section holding no
matching row SHALL be hidden. Filtering SHALL NOT change the stored collapse
state, so clearing the filter restores the panel to the shape the user left it
in.

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

### Requirement: The Resource panel is fully keyboard-operable

Every Resource panel action SHALL be reachable from the keyboard as well as the mouse
(`.claude/rules/keyboard-first.md`): moving between kinds, opening one, collapsing and
expanding a section, and focusing, typing into and clearing the filter. One selection
SHALL be moved by both clicks and the keyboard; hover SHALL NOT move it. The panel SHALL
show its keys in a hint row.

#### Scenario: Navigate and open from the keyboard

- **WHEN** the Resource panel has focus and the user presses Down twice, then Enter
- **THEN** the second visible kind below the current selection opens, as a double-click would open it

#### Scenario: Collapse and expand a section from the keyboard

- **WHEN** a kind in the Workloads section is selected and the user presses Left
- **THEN** Workloads collapses, and Right expands it again

#### Scenario: Filter from the keyboard

- **WHEN** the Resource panel has focus and the user presses `/`, types `ingr` and presses Escape
- **THEN** the filter field takes focus and narrows the list to matching kinds, and Escape clears it and returns focus to the list

### Requirement: Every discovered kind opens a list panel

Opening any kind from the Resource panel SHALL open a live list panel of that kind's objects for
its cluster context, and SHALL NOT open a placeholder. Every list panel SHALL provide the same
behavior this capability requires of the Pods table: live updates, namespace scoping, filtering
and sorting, resizable and reorderable columns, a plural title, a shared reference-counted watch,
and full keyboard operation. A cluster-scoped kind's list SHALL have no namespace selector and no
Namespace column.

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

Every list panel SHALL show Name and Age, and Namespace for a namespaced kind. A list of one of
the built-in kinds below SHALL also show that kind's columns, matching what `kubectl get` shows by
default:

| Kind | Additional columns |
|---|---|
| Deployment | Ready (ready/desired), Up-to-date, Available |
| ReplicaSet | Desired, Current, Ready |
| StatefulSet | Ready (ready/desired) |
| DaemonSet | Desired, Current, Ready, Up-to-date, Available |
| Job | Status, Completions, Duration |
| CronJob | Schedule, Suspend, Active, Last schedule |
| ConfigMap | Data (key count) |
| Secret | Type, Data (key count) |
| Service | Type, Cluster IP, External IP, Ports |
| Ingress | Class, Hosts, Address, Ports |
| Endpoints | Endpoints |
| EndpointSlice | Address type, Ports, Endpoints |
| NetworkPolicy | Pod selector |
| PersistentVolumeClaim | Status, Volume, Capacity, Access modes, Storage class |
| PersistentVolume | Capacity, Access modes, Reclaim policy, Status, Claim, Storage class |
| StorageClass | Provisioner, Reclaim policy, Volume binding mode |
| Node | Status, Roles, Version, Internal IP |
| Namespace | Status |
| ServiceAccount | Secrets (count) |
| RoleBinding, ClusterRoleBinding | Role, Subjects |

Any other kind SHALL show only the base columns. Pods keep the columns their existing
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

### Requirement: Opening a list row opens the object's detail

Activating a row in a list panel (Enter on the selected row, or double-click) SHALL open that
object's detail panel in the same cluster context: the pod-detail panel for a Pod, the
object-detail panel for any other kind. If that object's panel is already open, it SHALL be
focused instead.

#### Scenario: Opening a Deployment from its list

- **WHEN** the user selects a row in a Deployments list and presses Enter
- **THEN** the object-detail panel for that Deployment opens, showing its replicas, selector and
  conditions

#### Scenario: Opening an already-open object

- **WHEN** the user double-clicks a Service row whose detail panel is already open
- **THEN** the existing panel is focused and no second panel opens

### Requirement: Every list panel can describe the selected object

Every list panel SHALL bind the same object keys as the Pods table, scoped to that panel:
describe (`d`) opens the selected object's detail panel, and YAML (`y`) opens it showing its YAML.
Each SHALL be a registered command, reachable from the command palette, and shown in the panel's
hint row.

#### Scenario: Describing a Service from its list

- **WHEN** a Services list panel has focus with a row selected and the user presses `d`
- **THEN** that Service's detail panel opens, or is focused if already open

#### Scenario: YAML from a list

- **WHEN** a ConfigMaps list panel has focus with a row selected and the user presses `y`
- **THEN** that ConfigMap's detail panel opens showing its YAML

### Requirement: Arrow keys start selection in a focused list

When a list panel has focus and no row is selected, Down SHALL select the first visible row and Up
SHALL select the last, so the keyboard reaches the rows without tabbing into the table first.

#### Scenario: Down on a freshly focused list

- **WHEN** a list panel has just received focus with no row selected and the user presses Down
- **THEN** the first visible row is selected, and Enter opens it

### Requirement: Custom resource list tabs show the kind only

A list panel for a kind outside the built-in API groups SHALL title its tab and title bar with the
kind's plural display name only, and SHALL show the kind's API group in the tab's tooltip.

#### Scenario: A CRD list tab

- **WHEN** a list panel for `certificates.cert-manager.io` is open
- **THEN** its tab reads "Certificates", and hovering the tab shows `cert-manager.io`

### Requirement: Resource panel lists all discovered kinds by default

When a window has no saved layout for its connected cluster, the Resource panel SHALL
list every resource kind the cluster's API discovery reports, including CRDs, instead of
a fixed subset.

#### Scenario: Fresh connection shows full discovery

- **WHEN** a window connects to a cluster and has no saved layout for it
- **THEN** the Resource panel lists all resource kinds from that cluster's API discovery,
  including any CRDs it defines

#### Scenario: Cluster dropdown appears with more than one connection

- **WHEN** a window has more than one cluster connection open
- **THEN** the Resource panel shows a dropdown to choose which connection's resources
  the list reflects

#### Scenario: Cluster dropdown absent with one connection

- **WHEN** a window has exactly one cluster connection open
- **THEN** the Resource panel shows no cluster dropdown

### Requirement: Opening a resource kind opens a dockable panel

Selecting a resource kind in the Resource panel, by double-click or from its context
menu, SHALL open a dockable panel for that kind if one is not already open for the same
kind, cluster, and namespace scope.

#### Scenario: Double-click opens a panel

- **WHEN** the user double-clicks a resource kind in the Resource panel
- **THEN** a dockable panel for that kind opens in the workspace

#### Scenario: Context menu opens a panel

- **WHEN** the user selects a resource kind's context menu and chooses to open it
- **THEN** a dockable panel for that kind opens in the workspace, equivalently to
  double-click

#### Scenario: Re-selecting an open kind focuses it instead of duplicating

- **WHEN** the user selects a kind that already has a panel open for the same cluster
  and namespace scope
- **THEN** the existing panel is focused and no second panel is opened

#### Scenario: A kind with no implemented panel opens a placeholder

- **WHEN** the user selects a discovered kind that has no concrete panel implementation
- **THEN** a placeholder panel opens for it, rather than nothing happening and rather
  than the kind being absent from the Resource panel

### Requirement: Panel title bar identifies kind, cluster, and namespace

A resource panel's title bar SHALL show the resource kind, the cluster name when the
window has more than one connection open, a namespace picker when the kind is
namespaced, a controls menu, and a close button.

#### Scenario: Single cluster omits the cluster name

- **WHEN** a resource panel belongs to a window with exactly one cluster connection
- **THEN** its title bar shows the resource kind without a cluster name

#### Scenario: Multiple clusters show the cluster name

- **WHEN** a resource panel belongs to a window with more than one cluster connection
- **THEN** its title bar shows both the resource kind and that panel's cluster name

#### Scenario: Namespaced kind shows a namespace picker

- **WHEN** a resource panel shows a namespaced kind
- **THEN** its title bar includes a namespace picker

#### Scenario: Cluster-scoped kind omits the namespace picker

- **WHEN** a resource panel shows a cluster-scoped kind
- **THEN** its title bar shows no namespace picker

#### Scenario: Every resource panel has a controls menu and a close button

- **WHEN** any resource panel is open
- **THEN** its title bar shows both a controls menu and a close button

### Requirement: Resource kind navigation

A connected window SHALL provide a way to switch which resource kind its active panel
displays, without closing and reopening the window's connection. Which kinds are
offered is governed by the Resource panel above, not by this requirement: the
Resource panel's discovery-driven list is the single place that decides what the
window can show, so this requirement only covers switching without dropping the
connection.

#### Scenario: Switch from Pods to Logs

- **WHEN** a window is showing a Pods panel for a connected cluster
- **AND** the user navigates to Logs
- **THEN** the panel is replaced by a Logs view for the same connection, without
  reconnecting or losing the cluster session

### Requirement: The namespace scope indicator names a matching set

A namespaced panel's namespace scope indicator SHALL show a saved namespace set's name when the
panel's scope is an include set whose namespaces are exactly that set's namespaces. Otherwise the
indicator SHALL read as before, as a count of included namespaces or as "all namespaces". Because a
panel stores the namespaces it was switched to rather than a reference to the set, deleting or
editing a set SHALL leave an already-switched panel's scope and indicator unchanged until the panel is
switched to a set again.

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
as a link, each container's image and state, and its most recent Warning event if it has one. The
popover SHALL offer an Open Details action, by button and by Enter, that opens the pod's detail panel
and closes the popover. Space or Escape SHALL close it and return focus to the table.

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

### Requirement: Pod rows have a context menu
Right-clicking a pod row SHALL select it and open a context menu offering Quick Look, Open Details,
Logs and YAML, each running the same command as its key.

#### Scenario: Logs from the context menu
- **WHEN** the user right-clicks a pod row and chooses Logs
- **THEN** that pod's logs open, as pressing `l` on the row would

### Requirement: Filtering the namespace picker

A namespaced panel's namespace picker SHALL show a filter input when it opens, with keyboard focus
in it. Typing SHALL narrow the listed namespaces to those whose name contains the typed text,
ignoring case. The "All namespaces" entry SHALL remain listed first regardless of the filter.
Toggling a namespace SHALL keep the picker open with its filter text, so several namespaces can be
picked in a row. Up and Down SHALL move through the filtered entries, and Enter SHALL toggle the
highlighted one. Escape SHALL clear non-empty filter text, and close the picker when the text is
already empty. When the filter matches no namespace, the picker SHALL say so. The filter text
SHALL NOT survive closing the picker.

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
