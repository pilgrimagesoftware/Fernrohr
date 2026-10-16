# Proposal

## Why

Fernrohr's only exec path opens an interactive shell (`exec-terminal`). The command is hardcoded
to `bash || sh` in `App/app/src/k8s/resource/exec/transport.rs`. Users who run the same in-pod
commands over and over must retype them in a shell every time, and first find the right pod. A
common example is `nats stream info X` in a NATS box pod. App#296 asks for a library of saved
commands. Each entry is either general, so it runs in any pod, or tied to a namespace, a label
selector, a container, or a workload or Service, the way `kubectl exec deploy/x` resolves one.
The user can create an entry from a pod, workload or Service, with the dialog prefilled, and run
it straight away.

## What Changes

- **Exec library entries.** Each entry has a unique name, a command and a target:
  - **Any pod**: runs in whichever pod the user runs it from.
  - **Pod selector**: an optional namespace plus an optional label selector, in `kubectl -l`
    syntax.
  - **Workload**: a Deployment, StatefulSet, DaemonSet or Service, by namespace and name.

  An entry can also name a container. Entries are stored in `exec-library.toml` in the
  preferences dir. The file is hand-editable, written atomically, and kept on a parse error, with
  the error reported.
- **Running an entry** opens the existing shell panel. The panel runs the entry's command in place
  of the shell, with a TTY, so interactive and colored output work. When the command ends, the
  panel stays readable with its exit status, per `exec-terminal`. A restored panel comes back
  ended and never reruns the command.
- **Resolving the pod** works like kubectl:
  - **Pod selector**: the Running pods that match the namespace and selector.
  - **Workload**: the Running pods matched by the workload's own `spec.selector`.
  - **Any pod**, run from a pod: that pod.

  If exactly one pod matches, the entry runs there. If several match, a chooser lists them with
  the first one preselected, so Enter runs it. If none match, an error names the entry and the
  scope. The container is the entry's container. Failing that, it is the pod's
  `kubectl.kubernetes.io/default-container`, and failing that, the first container. A named
  container that the pod lacks is an error.
- **Exec Library window.** It lists, filters, runs, creates, edits, duplicates and deletes entries.
  Run uses the context of the panel the window was opened from. With no cluster panel behind it,
  Run asks which connected context to use.
- **From a resource.** Pod, Deployment, StatefulSet, DaemonSet and Service rows and detail panels
  gain two actions:
  - **Run from Library…** lists only the entries that apply to that resource and runs the chosen
    one there.
  - **Save to Library…** opens the entry dialog prefilled from the resource. For a Pod, the
    target is a pod selector built from the pod's namespace and its controller's selector, or its
    own labels minus the per-revision hash labels, plus its current container. For a workload or
    Service, the target is that workload. The dialog has **Save**, **Save and Run** and
    **Cancel**.
- Every action is a registered command, available in the command palette and keyboard-operable,
  with the keys shown.

## Capabilities

### New Capabilities
- `exec-command-library`: saved exec commands, their targets and pod resolution, the library
  window, the entry dialog, resource-scoped run and save actions, and persistence.

### Modified Capabilities
- None. The shell panel's existing requirements, such as ended sessions and the hint row, apply
  unchanged to a panel that runs a library command.

## Impact

- New: `App/app/src/config/exec_library.rs` (data plus validation), `App/app/src/ui/exec_library/`
  (store, window, entry dialog, chooser, commands), and a pod resolver shared with
  `port_forwarding::resolve_service` and `label_selector::of_object`.
- `ExecTarget` gains an optional command, which is saved with the panel so a restored panel can
  show its title. The transport takes the command in place of `SHELL` when one is set.
- Context menus and detail panels for Pods, Deployments, StatefulSets, DaemonSets and Services.
- `App/docs/site/latest/`: a new Exec Library page, plus links from the shell and resource pages.

## Non-goals

- Scoping an entry to particular kube contexts. Entries apply in every context.
- Running an entry in several pods at once, or capturing its output anywhere but the shell panel.
- Placeholders in commands. Commands are static; placeholders are tracked in Fernrohr-App#298.
- Agent (MCP) access to the library.
