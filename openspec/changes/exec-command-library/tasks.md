# Tasks

## 1. Library data and persistence

- [ ] 1.1 Add `config/exec_library.rs`: `ExecLibrary { entries }`, `Entry`, `Target` (D3 TOML
  shape), and create/update/duplicate/remove with the unique-name, command-split (`shell-words`),
  selector-parse and workload-field validation from D5, each with a typed error. Verify with unit
  tests for every validation error and a TOML round trip that preserves order.
- [ ] 1.2 Add the global store `ui/exec_library/store.rs` on `preference_dir()/exec-library.toml`
  with atomic writes, save-on-change and the read-only-on-parse-error behavior (D6). Wire it in
  `main.rs`. Verify with tests for the "Restart" and "Broken file" scenarios. Assert that the
  broken file's bytes are unchanged.

## 2. Pod resolution and running

- [ ] 2.1 Add `resolve_pods(client, &Target)` (D3) and the container rule: entry container, then
  `kubectl.kubernetes.io/default-container`, then the first container, with an error for a
  missing container. Refactor `port_forwarding::resolve_service` onto it. Verify against the mock
  cluster for 0, 1 and 3 matches, all-namespace selectors, Pending pods excluded, each container
  rule, and the existing port-forward tests still passing.
- [ ] 2.2 Add `command: Option<LibraryCommand>` to `ExecTarget` and its saved state, and run the
  command's argv in place of `SHELL` when it is set (D1). Show the entry name and pod in the
  title. Restore it as ended. Verify with a transport test that the attached argv is the entry's,
  and a restore test that no exec request is made.
- [ ] 2.3 Add the pod chooser and the connected-context chooser (D5). Verify with
  keystroke-driven tests: Enter runs the preselected first pod, and Esc opens nothing.

## 3. Exec Library window

- [ ] 3.1 Build the window (D5): filter, list, icon-button toolbar with tooltips, and a hint row.
  Register `exec_library.open/new/run/edit/duplicate/delete` with the irreversible Delete
  confirmation. Verify with keystroke tests for "Run from the keyboard" and "Delete needs a
  deliberate confirmation", plus tooltip assertions.
- [ ] 3.2 Build the entry dialog (D5): fields shown by target kind, inline errors, and Save,
  Save and Run, and Cancel with their keys. Verify with keystroke tests for "Create without the
  mouse", "Duplicate name" and "Unbalanced quote", and that focus returns on Escape.

## 4. Resource entry points

- [ ] 4.1 Add Run from Library… (`exec_library.run_from`) to Pod, Deployment, StatefulSet,
  DaemonSet and Service rows (context menu) and detail panels, with the applies-to filter (D3).
  Verify with the "Only matching entries offered" scenario and a workload-row case.
- [ ] 4.2 Add Save to Library… (`exec_library.save_from`) on the same surfaces, with the D4
  prefill. Verify with "Pod prefill drops hash labels", an owner-selector prefill when the
  ReplicaSet and Deployment are cached, and "Save and run from a Deployment".

## 5. Documentation

- [ ] 5.1 Add an Exec Library page under `docs/site/latest/`. It covers entries and targets, how a
  pod is picked, running from the window and from a resource, saving from a resource, the
  keyboard route, `sh -c` for pipes and variables, and that the file is plain text with no place
  for secrets. Link it from the shell and resource pages. Use placeholders only. Verify with the
  docs build and link check.
