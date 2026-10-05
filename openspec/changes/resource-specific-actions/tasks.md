# Tasks

## 1. Port Forward entry points for Pods and Services

`k9s-remaining-keybindings` already wired `K8sPortForwardConfig`/`PodPortForwardTransport`
(`forward/k8s/port_forward.rs`) into `ManagedForward`'s registry, resolves a Service to a ready
endpoint Pod, and lists/stops the result in Manage Tunnels' "Port forwards" section
(`PodsPanel::on_action_port_forward_pod`, `ObjectListPanel::on_action_port_forward_service`). None
of that is redone here - this task only adds the entry points a mouse user without the `shift-f`
keybinding has no way to reach today.

- [ ] 1.1 Add a Port Forward action to the Pods table row context menu (new - the table has no
  right-click menu yet) and the pod detail panel, each calling
  `on_action_port_forward_pod`'s existing handler, disabled when the pod isn't Running. Verify
  with a test asserting the action is absent/disabled for a Pending or Terminated pod.
- [ ] 1.2 Add the Port Forward action to the generic object list's row context menu (alongside its
  existing "Open" item) and the Service's object-detail panel section, each calling
  `on_action_port_forward_service`'s existing handler. Verify with a test covering the action's
  presence only on Service rows.

## 2. CronJob Trigger Now, and Job/CronJob Suspend/Resume

- [ ] 2.1 Add the shared mutating module (or extend `bulk_actions` if it's still under budget)
  with: a CronJob-to-Job creation call (building a `Job` from `spec.jobTemplate`, mirroring
  `kubectl create job --from=cronjob`), and a suspend/resume patch for `spec.suspend` on Job and
  CronJob. Verify with a test per call against the mock cluster.
- [ ] 2.2 Add Trigger Now to the CronJob row context menu and detail panel, behind a
  confirmation naming the CronJob. Verify with a keystroke-driven test: confirm, and assert a
  new Job exists with the CronJob's job template and the CronJob's own `spec` is unchanged.
- [ ] 2.3 Add Suspend/Resume to Job and CronJob rows and detail panels, offering whichever of
  the two applies to the object's current `spec.suspend`, behind a confirmation. Verify with a
  test toggling suspend on a CronJob and asserting the offered action flips to Resume.

## 3. Deployment Pause/Resume Rollout

- [ ] 3.1 Add a pause/resume patch for `spec.paused` on Deployment to the shared mutating module.
  Verify with a test asserting the field's value after each call.
- [ ] 3.2 Add Pause Rollout/Resume Rollout to Deployment rows and the Deployment's object-detail
  section only (not DaemonSet or StatefulSet), offering whichever applies to the current
  `spec.paused`, behind a confirmation. Verify with a test asserting the action is absent for
  DaemonSet and StatefulSet rows.

## 4. PersistentVolumeClaim Expand

- [ ] 4.1 Add an expand call patching `spec.resources.requests.storage` to the shared mutating
  module. Verify with a test asserting the field's value after the call, and a test asserting
  the call surfaces the cluster's own rejection message when the mock cluster returns one.
- [ ] 4.2 Add an Expand action to PersistentVolumeClaim rows and the object-detail section,
  prompting for a new size with inline client-side validation rejecting a size smaller than
  current, then a confirmation naming the old and new size. Verify with a test covering the
  client-side rejection and a test covering a successful patch.

## 5. Node taint/untaint

- [ ] 5.1 Add add-taint and remove-taint patches for `spec.taints` to the shared mutating module.
  Verify with a test adding a taint and asserting it appears in `spec.taints`, and a test
  removing one of two existing taints and asserting only the targeted one is gone.
- [ ] 5.2 Add an Add Taint action (key/value/effect prompt) and a per-existing-taint Remove
  action to Node rows and the Node's object-detail section, each behind a confirmation. Verify
  with a keystroke-driven test for both add and remove.

## 6. ServiceAccount Create Token

- [ ] 6.1 Add a `TokenRequest` call using `kube-client`'s `create_subresource("token", ...)` to
  the shared mutating module. Verify with a test against the mock cluster asserting a token
  comes back.
- [ ] 6.2 Add a Create Token action to ServiceAccount rows and the object-detail section,
  showing the result with the same reveal/hide/never-persisted handling Secret values already
  get (reusing that existing component rather than building a second one). Verify with a test
  asserting the token is hidden after the panel closes or Hide runs, and is never logged or
  written to disk.

## 7. Ingress Open Host in Browser

- [ ] 7.1 Add an Open Host in Browser action to Ingress rows and the object-detail section,
  reading the first rule's host from already-loaded data and opening `https://<host>` via the
  OS's default-browser call, with no confirmation. Verify with a test asserting the correct URL
  is constructed for an Ingress with multiple rules (uses the first) and one with none (action
  absent).

## 8. Keyboard and palette wiring

- [ ] 8.1 Register every new action from Tasks 1-7 as a command with a default binding and
  `keymap.toml` override, scoped to the relevant panel/row `KeyContext`, so each appears in the
  command palette while that row or panel has focus and in that row's context menu. Verify with
  a test per action asserting its palette entry appears only when a row/panel of the matching
  kind has focus.

## 9. Final checks

- [ ] 9.1 Run `cargo fmt && cargo clippy -- -D warnings && cargo test` and confirm all pass.
- [ ] 9.2 Manually exercise every action against a test cluster (`cargo run`): open a Port Forward
  from the Pod detail panel and from a Service row's context menu (the `shift-f` keyboard path is
  already covered by `k9s-remaining-keybindings`'s own manual checks), trigger a CronJob,
  suspend/resume a Job and a CronJob, pause/resume a Deployment rollout, expand a PVC (including a
  rejected expansion), add/remove a Node taint, create and reveal/hide a ServiceAccount token, and
  open an Ingress host in the browser.
