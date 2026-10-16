# Design

## Context

- Shell sessions: `ExecTarget {namespace, pod, container}` opens `NavTarget::Exec`. The panel runs
  `transport::SHELL` with a TTY and restores as `ExecPanel::ended`.
- Container choice for shells: `pods/shell.rs` (`running_containers`, a button dialog).
- Selector helpers: `k8s/label_selector.rs` (`parse`, `of_object`, matching).
- Service-to-pod: `port_forwarding::resolve_service`.
- Persistence template: `namespace-sets.toml`, made of `config/namespaces.rs` and
  `ui/namespace_sets/store.rs`. Atomic writes come from `config/saved_layouts.rs::write_atomic`.
- Form dialog template: the tunnel editor.

## Decisions

### D1: The command runs in the shell panel, with a TTY

`ExecTarget` gains `command: Option<LibraryCommand { entry_name, argv }>`. The transport passes
`argv` in place of `SHELL`, with the same AttachParams (stdin and TTY on). This reuses scrollback,
copy, bracketed paste, the ended notice and restore-as-ended, at no cost. Interactive tools such
as `nats` prompts or `top` behave as they do in a shell. The panel title is the entry name plus
the pod. Rejected: a separate non-TTY output panel. It duplicates the terminal's text handling
and breaks interactive commands.

### D2: Commands are argv, split with POSIX quoting, and never go through a shell

The command string is split the way command tunnels split theirs (`shell-words`), and is sent as
argv. That keeps `kubectl exec -- …` semantics and avoids depending on a shell inside the image.
To use pipes or `$VARS`, users write `sh -c '…'` themselves, and the docs say so. Splitting
errors are shown inline in the dialog and block Save.

### D3: Target model and pod resolution

```toml
[[entries]]
name = "NATS stream info"
command = "nats stream info X"
container = "nats-box"          # optional
[entries.target]
kind = "selector"               # "any" | "selector" | "workload"
namespace = "nats-system"       # optional for selector, required for workload
selector = "app.kubernetes.io/component=nats-box"
# workload: workload_kind = "Deployment" | "StatefulSet" | "DaemonSet" | "Service", name = "..."
```

One resolver, `resolve_pods(client, &Target) -> Vec<Pod>`:
- `selector`: list pods (namespaced, or all namespaces when no namespace is set) with the selector
  as a server-side `labelSelector`.
- `workload`: get the object, take `label_selector::of_object`, then list as for `selector`.
  `resolve_service` is refactored onto this.

The resolver keeps only Running pods and sorts them by namespace, then name, so the order is
deterministic. 0 pods gives an error, 1 runs, and 2 or more open the chooser (D5).

**Applies-to** for Run from Library on a resource:
- Pod: `any` entries; `selector` entries whose namespace matches (or is unset) and whose selector
  matches the pod's labels; `workload` entries whose workload selects the pod, checked against
  cached objects only, with no extra request.
- Workload or Service: `any` entries, plus `selector` and `workload` entries that match it or one
  of its pods.

A resolved pod still runs only if it is Running.

### D4: Prefill from a resource

- Pod: `selector` target, the pod's namespace, and the selector of its controlling owner (found
  through `ownerReferences`, then ReplicaSet to Deployment) when that owner is cached. Otherwise
  the pod's labels minus `pod-template-hash`, `controller-revision-hash`,
  `statefulset.kubernetes.io/pod-name` and `apps.kubernetes.io/pod-index`. The container is the
  one selected in the pod detail panel, else empty.
- Deployment, StatefulSet, DaemonSet or Service: a `workload` target with its kind, namespace and
  name.

Every prefilled field stays editable, including the target kind.

### D5: Windows and dialogs, keyboard first

- **Exec Library window**: one window, like Settings. A filter input sits above a list, with
  Enter to Run, `e` to Edit, `n` for New, `d` to Duplicate and `cmd-backspace` to Delete. Delete
  is an irreversible confirmation, so focus starts on Cancel. The window's toolbar controls are
  icon buttons with tooltips that show the live key (`icon-buttons.md`), and a hint row lists the
  keys.
- **Entry dialog**: the fields are Name, Command, Target kind (a segmented control, each segment a
  tab stop), Namespace, Selector or Workload kind plus Name (shown by target kind), and Container.
  The buttons are Save (Enter), Save and Run (`cmd-enter`) and Cancel (Esc), each showing its
  key. Tab moves between fields and does not indent. Validation errors show inline:
  - the name is required and must be unique, ignoring case and surrounding spaces;
  - the command must be non-empty and split cleanly;
  - the selector must parse;
  - a workload target needs a namespace and a name.
- **Pod chooser**: a picker dialog with the first pod preselected. Enter runs, Esc cancels.
- **Context chooser** for Run with no cluster panel: the same picker over connected contexts.
- The commands are registered in the central registry:
  - `exec_library.open`, `exec_library.new`, `exec_library.run`, `exec_library.edit`,
    `exec_library.duplicate`, `exec_library.delete`;
  - `exec_library.run_from` and `exec_library.save_from`, scoped to the Pod, workload and Service
    row and detail key contexts, also in their row context menus.

### D6: Persistence

`preference_dir()/exec-library.toml` holds `#[serde(default)] struct ExecLibrary { entries:
Vec<Entry> }`, kept in file order. A global store follows the namespace-sets pattern: it saves
on every change, observers are notified, and writes are atomic. On a parse error the file is kept
untouched, the library opens empty and read-only, and a banner names the file and the error. That
avoids overwriting a hand edit with a typo in it. The file is plain text, and the docs warn
against putting secrets in commands.

## Risks

- Applies-to checks for `workload` entries on a Pod rely on cached workload objects. With no
  watcher on that kind, such entries are not offered on the pod, but they still run from the
  library or from the workload. That's acceptable, and it is documented.
- An all-namespaces selector list can be large on big clusters. It is one request per run, no
  watcher, and it is filtered on the server by `labelSelector`.
