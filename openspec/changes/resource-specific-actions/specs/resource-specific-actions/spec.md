# Spec Delta

## Purpose

Gives individual Kubernetes object kinds the actions specific to them - starting a port forward,
triggering or suspending a workload, pausing a rollout, expanding storage, tainting a node,
minting a service account token, opening an ingress host - each reachable from that kind's list
row, detail panel, and the command palette, alongside the generic cross-kind actions
`bulk-object-actions` already provides.

## ADDED Requirements

### Requirement: Port Forward for Pods and Services

A Pod or Service row and its detail panel SHALL offer a Port Forward action that starts a
`ManagedForward` to that object, prompting for the local port to use and defaulting to the
object's first exposed port. The forward SHALL appear wherever the app already lists active
forwards, and SHALL be stoppable from there the same way an SSH tunnel is.

#### Scenario: Starting a port forward from a Pod

- **WHEN** the user chooses Port Forward on a running Pod and accepts the default local port
- **THEN** a `ManagedForward` to that Pod starts and appears in the active-forwards list

#### Scenario: Port Forward is not offered for a Pod that isn't running

- **WHEN** a Pod is Pending or Terminated
- **THEN** its Port Forward action is disabled or absent

### Requirement: Trigger Now for CronJobs

A CronJob row and its detail panel SHALL offer a Trigger Now action that creates a Job from the
CronJob's spec immediately, the same as `kubectl create job --from=cronjob/<name>`, and SHALL
show a confirmation naming the CronJob before creating the Job.

#### Scenario: Triggering a CronJob early

- **WHEN** the user chooses Trigger Now on a CronJob and confirms
- **THEN** a new Job is created from that CronJob's job template, and the CronJob's own schedule
  is unaffected

### Requirement: Suspend and Resume for Jobs and CronJobs

A Job or CronJob row and its detail panel SHALL offer a Suspend action when not already
suspended and a Resume action when suspended, toggling `spec.suspend`, and SHALL show a
confirmation before suspending or resuming.

#### Scenario: Suspending a running CronJob

- **WHEN** the user chooses Suspend on an active CronJob and confirms
- **THEN** `spec.suspend` is set `true` and the action offered for that row becomes Resume

### Requirement: Pause and Resume Rollout for Deployments

A Deployment row and its detail panel SHALL offer a Pause Rollout action when not already
paused and a Resume Rollout action when paused, toggling `spec.paused`, and SHALL show a
confirmation before pausing or resuming. This action SHALL NOT be offered for DaemonSet or
StatefulSet rows, which have no paused-rollout concept.

#### Scenario: Pausing a Deployment's rollout

- **WHEN** the user chooses Pause Rollout on a Deployment mid-rollout and confirms
- **THEN** `spec.paused` is set `true` and the rollout stops progressing

### Requirement: Expand for PersistentVolumeClaims

A PersistentVolumeClaim row and its detail panel SHALL offer an Expand action that prompts for a
new storage size no smaller than the claim's current size, patches
`spec.resources.requests.storage` to that value, and SHALL show a confirmation naming the old and
new size before patching. A cluster rejection (e.g. the StorageClass does not allow expansion)
SHALL be shown as the cluster's own error rather than a generic failure.

#### Scenario: Expanding a claim's storage

- **WHEN** the user chooses Expand on a 10Gi claim, enters 20Gi, and confirms
- **THEN** the claim's `spec.resources.requests.storage` is patched to 20Gi

#### Scenario: Expansion rejected by the StorageClass

- **WHEN** the claim's StorageClass does not allow volume expansion
- **THEN** the panel shows the cluster's rejection message rather than succeeding silently

### Requirement: Taint and untaint for Nodes

A Node row and its detail panel SHALL offer an Add Taint action that prompts for a key, an
optional value, and an effect (`NoSchedule`, `PreferNoSchedule`, or `NoExecute`), and SHALL list
the Node's existing taints each with its own Remove action. Both adding and removing a taint
SHALL show a confirmation before patching.

#### Scenario: Adding a taint

- **WHEN** the user adds a taint with key `dedicated`, value `gpu`, effect `NoSchedule`, and
  confirms
- **THEN** that taint appears in the Node's `spec.taints`

#### Scenario: Removing one of several taints

- **WHEN** a Node has two taints and the user removes one and confirms
- **THEN** only the removed taint is gone from `spec.taints`; the other remains

### Requirement: Create Token for ServiceAccounts

A ServiceAccount row and its detail panel SHALL offer a Create Token action that requests a
`TokenRequest` and shows the resulting token once. The token SHALL be revealed and hidden the
same way a Secret value is - not written to disk, logged, or placed in a window title, tab name,
palette entry or notification - and SHALL NOT be shown again once hidden; a new token requires
choosing Create Token again.

#### Scenario: Viewing a newly created token

- **WHEN** the user chooses Create Token on a ServiceAccount
- **THEN** the token is shown once, with a control to hide it, and is never written to disk or
  logged

### Requirement: Open Host in Browser for Ingresses

An Ingress row and its detail panel SHALL offer an Open Host in Browser action that opens the
first rule's host over https in the system's default browser. This action makes no cluster call
and requires no confirmation.

#### Scenario: Opening an Ingress's host

- **WHEN** the user chooses Open Host in Browser on an Ingress with host `app.example.com`
- **THEN** `https://app.example.com` opens in the system's default browser

### Requirement: Resource-specific actions are keyboard-reachable

Every action in this capability SHALL be reachable from the keyboard as well as the mouse, SHALL
be a registered command with a default binding and a `keymap.toml` override, and SHALL appear in
the command palette while the relevant row or detail panel has focus, and in that row's context
menu.

#### Scenario: Running a resource-specific action without a mouse

- **WHEN** the user navigates to a CronJob row with the keyboard and invokes Trigger Now by its
  command
- **THEN** the confirmation and the action are both reachable and operable without touching the
  mouse
