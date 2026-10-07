# Spec Delta

## MODIFIED Requirements

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

Any other kind SHALL show only the base columns, unless the panel has observed that kind's objects
carrying a `status.conditions` array, in which case it SHALL also show a Message column (see
"A condition-derived Message column for kinds outside the table" below). Pods keep the columns
their existing requirement defines.

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

## ADDED Requirements

### Requirement: A Ready column for kinds outside the table

A list panel for a kind that has no entry in the built-in columns table SHALL show a Ready column,
placed before the Message column, once the panel has observed at least one of that kind's objects
carrying a `Ready` condition. The cell SHALL show the condition's status (`True`, `False` or
`Unknown`), with a readiness indicator and tone matching the existing workload Ready cells: Good
for `True`, Bad for `False`, Warning for `Unknown`. An object of that kind without a `Ready`
condition SHALL show an empty cell. A kind none of whose loaded objects has a `Ready` condition
SHALL show no Ready column. The column SHALL be resizable, reorderable, and sortable, sorting
`False` before `Unknown` before `True` when ascending.

#### Scenario: A Kustomization list shows Ready

- **WHEN** a Kustomization list panel is open and one Kustomization's `Ready` condition is
  `"False"` while another's is `"True"`
- **THEN** the first row's Ready cell reads False in its Bad tone, and the second reads True in
  its Good tone

#### Scenario: A CRD with conditions but no Ready condition shows no Ready column

- **WHEN** a list panel is open for a CRD kind whose objects carry conditions, none of type
  `Ready`
- **THEN** the list shows a Message column but no Ready column

#### Scenario: Sorting by Ready surfaces failures first

- **WHEN** the user sorts a Kustomization list by Ready ascending
- **THEN** rows whose Ready is False come first, then Unknown, then True

### Requirement: A condition-derived Message column for kinds outside the table

A list panel for a kind that has no entry in the built-in columns table SHALL show a Message
column once the panel has observed at least one of that kind's objects carrying a non-empty
`status.conditions` array. The cell SHALL show the `Ready` condition's message when that condition
is present and its message is non-empty, toned Good for `True`, Bad for `False`, and Warning for
any other status. When no `Ready` condition is present, the cell SHALL instead show the most
recently transitioned condition that has a non-empty message, untoned. A kind whose objects never
carry conditions, or whose panel has no objects loaded yet, SHALL show no Message column. A kind
already covered by the built-in columns table SHALL NOT also gain a Message column.

The column SHALL behave like any other: resizable, reorderable, and sortable by its text.

#### Scenario: A Kustomization list shows its Ready message

- **WHEN** a Kustomization list panel is open and a Kustomization's `Ready` condition has
  `status: "False"` and a message explaining the failed reconciliation
- **THEN** that row's Message cell shows the failure message in its Bad tone

#### Scenario: A healthy HelmRelease shows a Good-toned message

- **WHEN** a HelmRelease's `Ready` condition has `status: "True"` and a message such as "Release
  reconciliation succeeded"
- **THEN** that row's Message cell shows that message in its Good tone

#### Scenario: No Ready condition falls back to the latest transition

- **WHEN** a CRD object has no `Ready` condition but has another condition with a non-empty
  message and the most recent `lastTransitionTime`
- **THEN** that row's Message cell shows that condition's message, untoned

#### Scenario: A CRD with no conditions shows no Message column

- **WHEN** a list panel is open for a CRD kind whose objects never set `status.conditions`
- **THEN** the list shows only the base columns, with no Message column

#### Scenario: A built-in kind with its own status column is unaffected

- **WHEN** a Jobs list panel is open
- **THEN** it keeps its existing Status, Completions and Duration columns and does not also gain a
  separate Message column

#### Scenario: Opening the object's detail reaches the full message

- **WHEN** the user selects a row whose Message cell is truncated and presses the describe key
- **THEN** that object's detail panel opens showing the full, untruncated message
