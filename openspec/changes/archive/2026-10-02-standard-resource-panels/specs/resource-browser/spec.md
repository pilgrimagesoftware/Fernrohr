# Spec Delta

## ADDED Requirements

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
