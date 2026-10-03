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
| Node | Status, Roles, Version, Internal IP, CPU, Memory, Disk, Pods |
| Namespace | Status |
| ServiceAccount | Secrets (count) |
| RoleBinding, ClusterRoleBinding | Role, Subjects |

Any other kind SHALL show only the base columns. Pods keep the columns their existing
requirement defines. The Node kind's CPU, Memory and Disk columns show current usage against
capacity, sourced from the cluster's metrics source (per `cluster-metrics`); its Pods column
shows the count of pods scheduled to it, sourced from the pods list rather than metrics.

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

#### Scenario: Node usage against capacity

- **WHEN** the Nodes table is open against a cluster with a detected metrics source
- **THEN** each node's row shows its current CPU and memory usage against that node's capacity,
  its disk size, and the number of pods scheduled to it

#### Scenario: Sorting Nodes by a resource column

- **WHEN** the user sorts the Nodes table by the CPU column
- **THEN** rows order by current CPU usage, ascending or descending per the existing sort
  convention

#### Scenario: No metrics source

- **WHEN** the Nodes table is open against a cluster with no detected metrics source
- **THEN** the CPU and memory columns show that usage is unavailable, distinct from a node
  reporting zero usage

#### Scenario: Pod count does not depend on metrics

- **WHEN** the Nodes table is open against a cluster with no detected metrics source
- **THEN** the pod-count column still shows the correct count, since it comes from the pods list
  rather than the metrics source
