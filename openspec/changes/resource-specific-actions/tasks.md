# Tasks

## 1. Port Forward for Pods

- [ ] 1.1 Wire `K8sPortForwardConfig`/`PodPortForwardTransport` (`forward/k8s/port_forward.rs`)
  into `ManagedForward`'s existing registry/supervisor so starting one follows the same
  reference-counted lifecycle an SSH tunnel already does. Verify with a test starting a forward
  to a mock cluster's Pod and asserting it registers and health-checks like an existing tunnel.
- [ ] 1.2 Add a Port Forward action to the Pods table row context menu and the pod detail panel,
  prompting for a local port and defaulting to the pod's first container port, disabled when the
  pod isn't Running. Verify with a test asserting the action is absent/disabled for a Pending or
  Terminated pod.
- [ ] 1.3 Surface Pod port forwards in the tunnels editor's forward list alongside SSH tunnels,
  with a stop control. Verify with a test asserting a started forward appears there and stopping
  it there tears it down.

## 2. Port Forward for Services

- [ ] 2.1 Add a Service resolution step: given a Service, list its ready endpoint Pods (via
  `Endpoints`/`EndpointSlice`) and pick one, reusing Task 1's Pod-forward path underneath.
  Verify with a test asserting the forward targets a ready endpoint and a second test asserting
  a Service with no ready endpoints reports that clearly instead of forwarding to nothing.
- [ ] 2.2 Add the Port Forward action to the generic object list's row context menu and the
  Service's object-detail panel section, defaulting the local port to the Service's first port.
  Verify with a test covering the action's presence only on Service rows.

## 3. CronJob Trigger Now, and Job/CronJob Suspend/Resume

- [ ] 3.1 Add the shared mutating module (or extend `bulk_actions` if it's still under budget)
  with: a CronJob-to-Job creation call (building a `Job` from `spec.jobTemplate`, mirroring
  `kubectl create job --from=cronjob`), and a suspend/resume patch for `spec.suspend` on Job and
  CronJob. Verify with a test per call against the mock cluster.
- [ ] 3.2 Add Trigger Now to the CronJob row context menu and detail panel, behind a
  confirmation naming the CronJob. Verify with a keystroke-driven test: confirm, and assert a
  new Job exists with the CronJob's job template and the CronJob's own `spec` is unchanged.
- [ ] 3.3 Add Suspend/Resume to Job and CronJob rows and detail panels, offering whichever of
  the two applies to the object's current `spec.suspend`, behind a confirmation. Verify with a
  test toggling suspend on a CronJob and asserting the offered action flips to Resume.

## 4. Deployment Pause/Resume Rollout

- [ ] 4.1 Add a pause/resume patch for `spec.paused` on Deployment to the shared mutating module.
  Verify with a test asserting the field's value after each call.
- [ ] 4.2 Add Pause Rollout/Resume Rollout to Deployment rows and the Deployment's object-detail
  section only (not DaemonSet or StatefulSet), offering whichever applies to the current
  `spec.paused`, behind a confirmation. Verify with a test asserting the action is absent for
  DaemonSet and StatefulSet rows.

## 5. PersistentVolumeClaim Expand

- [ ] 5.1 Add an expand call patching `spec.resources.requests.storage` to the shared mutating
  module. Verify with a test asserting the field's value after the call, and a test asserting
  the call surfaces the cluster's own rejection message when the mock cluster returns one.
- [ ] 5.2 Add an Expand action to PersistentVolumeClaim rows and the object-detail section,
  prompting for a new size with inline client-side validation rejecting a size smaller than
  current, then a confirmation naming the old and new size. Verify with a test covering the
  client-side rejection and a test covering a successful patch.

## 6. Node taint/untaint

- [ ] 6.1 Add add-taint and remove-taint patches for `spec.taints` to the shared mutating module.
  Verify with a test adding a taint and asserting it appears in `spec.taints`, and a test
  removing one of two existing taints and asserting only the targeted one is gone.
- [ ] 6.2 Add an Add Taint action (key/value/effect prompt) and a per-existing-taint Remove
  action to Node rows and the Node's object-detail section, each behind a confirmation. Verify
  with a keystroke-driven test for both add and remove.

## 7. ServiceAccount Create Token

- [ ] 7.1 Add a `TokenRequest` call using `kube-client`'s `create_subresource("token", ...)` to
  the shared mutating module. Verify with a test against the mock cluster asserting a token
  comes back.
- [ ] 7.2 Add a Create Token action to ServiceAccount rows and the object-detail section,
  showing the result with the same reveal/hide/never-persisted handling Secret values already
  get (reusing that existing component rather than building a second one). Verify with a test
  asserting the token is hidden after the panel closes or Hide runs, and is never logged or
  written to disk.

## 8. Ingress Open Host in Browser

- [ ] 8.1 Add an Open Host in Browser action to Ingress rows and the object-detail section,
  reading the first rule's host from already-loaded data and opening `https://<host>` via the
  OS's default-browser call, with no confirmation. Verify with a test asserting the correct URL
  is constructed for an Ingress with multiple rules (uses the first) and one with none (action
  absent).

## 9. Keyboard and palette wiring

- [ ] 9.1 Register every new action from Tasks 1-8 as a command with a default binding and
  `keymap.toml` override, scoped to the relevant panel/row `KeyContext`, so each appears in the
  command palette while that row or panel has focus and in that row's context menu. Verify with
  a test per action asserting its palette entry appears only when a row/panel of the matching
  kind has focus.

## 10. Final checks

- [ ] 10.1 Run `cargo fmt && cargo clippy -- -D warnings && cargo test` and confirm all pass.
- [ ] 10.2 Manually exercise every action against a test cluster (`cargo run`): start and stop a
  Pod port forward and a Service port forward, trigger a CronJob, suspend/resume a Job and a
  CronJob, pause/resume a Deployment rollout, expand a PVC (including a rejected expansion),
  add/remove a Node taint, create and reveal/hide a ServiceAccount token, and open an Ingress
  host in the browser.
