# Proposal

## Why

Fernrohr's command registry covers navigation, panel/tab management, and read-only resource
inspection (describe, logs, YAML), but several of k9s's highest-value keyboard actions for acting
on a resource - not just viewing it - have no equivalent: there is no way to delete or force-kill a
Pod, shell into a container, edit a resource's YAML, start a port-forward from the Pods panel, view
a crashed container's previous logs, or jump straight to a namespace by number. Users coming from
k9s expect these actions to exist and to be one keystroke away.

## What Changes

- Add a `resource-actions` command set on resource-browser rows: delete (with confirmation), force
  kill (skip confirmation, `ctrl-d`/`ctrl-k`-style), edit resource YAML (opens an editable YAML view,
  applies on save), and shell/exec into a Pod's container (opens an interactive exec session in a
  panel). **BREAKING**: none - purely additive commands.
- Add "previous logs" to the Pod logs panel: toggle between the running container's current log
  stream and its last-terminated instance's log output (k9s's `p`), for Pods that have restarted.
- Add "port-forward this resource" as a Pods/Services panel row command that starts a
  `managed-forward` Kubernetes port-forward and surfaces it in the existing tunnel-management UI,
  instead of requiring the user to configure one by hand.
- Add numeric quick-jump to a namespace (k9s's `0`-`9` against the namespace list) as a resource
  panel command, scoped to the panel's current namespace list ordering.
- Add a global "key hints" help overlay command (`?`) that lists every command available in the
  current context with its binding, sourced from the existing command registry.

## Capabilities

### New Capabilities
- `resource-actions`: delete, force-kill, edit-YAML, and shell-exec commands on a resource-browser
  row, each gated by resource kind and by whether the action applies (e.g. shell-exec only for Pods
  with a running container).
- `help-overlay`: a command that opens a context-aware list of every currently available command and
  its key binding, sourced from the command registry.

### Modified Capabilities
- `pod-logs`: add a "previous logs" mode that streams a Pod's last-terminated container instance
  instead of its current one.
- `managed-forward`: add a user-initiated path to start a Kubernetes port-forward from a
  resource-browser row (acquire) and to find/stop it from the existing tunnel management UI
  (release), rather than only tunnels configured ahead of time.
- `resource-browser`: add a numeric quick-jump command that selects a namespace by its position in
  the panel's current namespace list.

## Impact

- `App/app/src/k8s/resource/pods/commands.rs` and equivalent `commands.rs` modules for other
  resource kinds: new command registrations (delete, kill, edit, exec, port-forward).
- `App/app/src/k8s/resource/pod_detail/` or a new `exec` panel module: interactive shell/exec view.
- `App/app/src/k8s/resource/object_detail/`: editable YAML mode and apply-on-save.
- `App/app/src/util/shell/` or `k8s/resource/pod_logs/`: previous-logs toggle and source switch.
- `App/app/src/ui/tunnels/`: wiring a panel-initiated port-forward into the existing managed-forward
  list.
- `App/app/src/command.rs` / `keymap.rs`: new command ids and default bindings; no changes to the
  registry mechanism itself.
- New global help-overlay view, reading the command registry (`App/app/src/command.rs`).
- kube-rs: needs exec (`AttachParams`/`attach`) and port-forward API usage already proven by
  `managed-forward`'s Kubernetes implementation; delete/patch via the typed API.
