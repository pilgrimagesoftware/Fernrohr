# Tasks

## 1. Delete and force-kill

- [x] 1.1 Add a shared `resource_actions::delete(client, kind, name, namespace, force: bool)` helper
      that issues a typed delete, with `force` setting zero grace period, and verify a unit test
      covers both the normal and forced request shapes sent to a mocked client
- [x] 1.2 Register `pods.delete` (confirmation dialog, reuses existing confirm-dialog component) and
      `pods.kill` (no confirmation) commands on the Pods panel, and verify a test confirms `kill`
      skips the dialog while `delete` requires confirming
- [x] 1.3 Wire delete/kill failure into the existing panel error-surface path and verify a test shows
      a simulated API rejection renders a failure reason without removing the row

## 2. Edit resource YAML

- [x] 2.1 Add an editable mode to the existing `object_detail` YAML view (toggle via `object_detail.edit`
      command) and verify a test confirms the YAML text becomes editable and reverts on cancel
- [x] 2.2 Implement save as a server-side apply (`Patch::Apply`) of the edited manifest and verify a
      unit test confirms a well-formed edit is sent as an apply patch to a mocked client
- [x] 2.3 Validate YAML parses as a manifest for the resource's kind before allowing save, and verify
      a test confirms invalid YAML blocks save with a reason and does not call the client
- [x] 2.4 Surface apply conflicts (e.g. `managedFields` ownership conflicts, stale resourceVersion) as
      a failure that keeps the user's edited content, and verify a test simulates a conflict response
      and confirms the edit view retains its content
- [x] 2.5 Start edit from a list panel's selected row (Pods and every `ObjectListPanel`) and from a
      pod's detail panel, opening the object panel's edit view through one window action
      (`EditListedObject`). Every kind but Secret is editable, and a Secret says why. Verify
      real-window keystroke tests: `e` on a Deployment row, a Pods row and in a pod's detail panel
      each open the editor, and `e` on a Secret row gives the reason
      (pilgrimagesoftware/Fernrohr-App#134, #140).

## 3. Shell / exec into a container

- [x] 3.1 Add an exec session type that opens a kube-rs `AttachParams` stream (stdin writer + stdout/
      stderr reader) on a background tokio task, bridged to a GPUI Entity via bounded channels, and
      verify a unit test exercises the bridging logic against a mocked attach stream
- [x] 3.2 Add a `pods.shell` command, available only when the target Pod has a running container, and
      a panel view that renders the session's output and forwards keystrokes to the stdin writer, and
      verify a test confirms the command is absent from the palette for a Pod with no running container
- [x] 3.3 For a multi-container Pod, prompt for which container to shell into before opening the
      session, and verify a test covers the container-picker path versus the single-container
      shortcut
- [x] 3.4 Detect the exec stream ending (container exit or delete) and mark the session panel ended
      while retaining its transcript, and verify a test simulates stream closure and asserts the
      transcript is still readable afterward

## 4. Port-forward from a resource row

- [x] 4.1 Add a `pods.port_forward` (and equivalent Services row) command that builds a Kubernetes
      managed-forward acquire request for the selected resource and target port, reusing
      `managed_forward::acquire`, and verify a test confirms the request shape matches what the
      tunnel-management UI sends for a hand-configured forward
- [x] 4.2 When the target exposes more than one port, prompt for which port before acquiring, and
      verify a test covers both the single-port shortcut and the multi-port prompt
- [x] 4.3 Confirm the acquired forward appears in the existing tunnel-management list and that
      stopping it there releases it through the normal reference-counted path, and verify an
      integration-style test (mocked k8s port-forward) covers acquire-then-release end to end

## 5. Previous container logs

- [x] 5.1 Add a "previous logs" toggle command on the log panel that switches the stream source to
      the container's last-terminated instance (`previous: true` on the log request) and verify a
      test confirms the request flag flips with the toggle
- [x] 5.2 Handle the no-previous-instance case by showing an explanatory state instead of an empty
      view, and verify a test simulates a container with zero restarts and asserts that state renders
- [x] 5.3 Verify toggling back to current logs resumes streaming and includes lines emitted while
      previous logs was shown, via a test that interleaves the two streams in a mocked client

## 6. Namespace quick-jump

- [x] 6.1 Add quick-jump commands (bound to a modifier + digit, scoped to the resource panel's
      `KeyContext` so they don't collide with `tab.select_1`-`9`) that select the namespace at that
      position in the panel's current namespace list, and verify a test confirms position `n` selects
      the `n`th namespace in a fixture list
- [x] 6.2 Make an out-of-range position a no-op, and verify a test confirms the namespace selection is
      unchanged when invoked beyond the list's length
- [x] 6.3 Run the existing keymap conflict checker against the new default bindings and verify it
      reports no conflicts with current commands

## 7. Help overlay

- [x] 7.1 Add a `global.show_key_hints` command (default binding `?`) that opens an overlay built from
      the `CommandRegistry` filtered to the currently focused `KeyContext`, and verify a test confirms
      the listed commands match what the palette would show in that same context
- [x] 7.2 Show commands with no current binding as unbound rather than omitting them, and verify a
      test covers a command whose binding was removed via the keybindings editor
- [x] 7.3 Verify dismissing the overlay returns keyboard focus to its prior location via a focus-order
      test

## 8. Command registry and keymap wiring

- [x] 8.1 Register all new command ids and default bindings (`pods.delete`, `pods.kill`,
      `object_detail.edit`, `pods.shell`, `pods.port_forward`, resource-panel quick-jump commands,
      `global.show_key_hints`) and verify `keymap.toml`'s first-run defaults include every new id
- [x] 8.2 Add palette and keybindings-editor coverage for each new command by relying on the existing
      registry-driven rendering, and verify a test confirms each new command id appears in both
      without additional per-command UI code

## 9. Manual checks (need a person, a real keyboard and a cluster)

- [x] 9.1 Delete and kill: on a throwaway pod, `ctrl-d` asks "Delete Pod?" - Cancel leaves it,
      Delete removes the row once the cluster drops the pod; `ctrl-k` removes it at once. With an
      account that may not delete pods, the refusal shows above the table and the row stays.
- [x] 9.2 Edit YAML: open a Deployment, `e`, change `spec.replicas`, `cmd-s` - the panel shows the new
      count. Edit again, change the object elsewhere (e.g. `kubectl scale`), then `cmd-s` - the
      conflict shows and the edited text stays. `e` on a Secret says it can't be edited here.
- [x] 9.3 Shell: `s` on a running single-container pod opens a shell panel; `ls` + Enter lists files;
      `exit` + Enter marks the session ended with the transcript kept. On a two-container pod, `s`
      asks which container. On a pod with nothing running, `s` does nothing and the palette doesn't
      offer it.
- [x] 9.4 Port-forward: `shift-f` on a pod declaring one port prints where it listens; `curl` that
      address reaches the pod. On a Service row, the chosen port reaches a pod behind it. Manage
      Tunnels (`cmd-shift-t`) lists the forward as Up; Stop removes it and the local port closes.
- [x] 9.5 Previous logs: on a container that has restarted, `p` in the Logs panel shows the previous
      instance's log and the title reads "(previous)"; `p` again returns to the live log. On a
      container that never restarted, `p` says there is no previous instance.
- [x] 9.6 Keys on a real keyboard: `?` (shift-/ on a US layout) opens Key Hints outside a text field
      and types `?` inside one; `alt-1`..`alt-9` and `alt-0` jump namespaces with Option held on
      macOS (Option-digit types a symbol in text fields, so check it doesn't fire there).
- [x] 9.7 Delete everywhere: in a Secrets list, `ctrl-d` on a throwaway Secret asks with its name
      quoted in the accent colour and both buttons showing their keys; Enter deletes it. In a pod's
      detail panel, `s` opens a shell, `shift-f` forwards, `ctrl-d` asks and the panel then shows the
      pod deleted. A metrics kind's list offers neither Delete nor Edit.

## 10. Delete everywhere and confirmation styling (#136, #143)

- [x] 10.1 Record the `delete` and `patch` verbs from discovery, and verify a test reads both from a
      fixture, including a get/list-only kind that has neither
- [x] 10.2 Make delete one reusable flow (`delete_flow`: confirm, send, report through a callback,
      never inside the caller's update) that the Pods panel uses and bulk delete can build on, and
      verify unit tests cover the question's wording and a disconnected context's deferred refusal
- [x] 10.3 Offer `ctrl-d` on every list row whose kind lists `delete`, outside text fields, and verify
      a real-window test deletes a Secret from the Secrets list (Escape cancels, Enter deletes, the
      row goes), shows a refusal, and offers nothing for a kind without `delete` or in the filter
- [x] 10.4 Offer `ctrl-d` in the object detail panel while its object is loaded, present and deletable,
      and verify a real-window test deletes it and the panel then shows it deleted
- [x] 10.5 Offer `ctrl-d`, `ctrl-k`, `s` (while a container runs) and `shift-f` in the pod detail panel
      on its own pod, reusing the Pods list's actions and flows, with Shell, Port forward and Delete
      hints after Logs, and verify real-window keystroke tests for each
- [x] 10.6 Set object names apart in every confirmation (quoted, code font, accent colour), and verify
      a render test shows a name drawn as its own code-font run in the accent colour
- [x] 10.7 Show each confirmation button's bound key (Enter, Escape) from the live keymap and make
      Enter confirm, and verify a render test draws both keys and keystroke tests confirm and cancel
- [x] 10.8 Offer Edit only for a kind that lists `patch`, and verify a real-window test shows a
      read-only kind's row offers no Edit, and a test finds no clash for any Edit key
- [x] 10.9 Have the palette read every key context on the focus path, not only each element's
      primary one, and verify a test offers a command gated to a secondary context

## Notes

- Implemented in pilgrimagesoftware/Fernrohr-App#122. Deviations worth a look:
  - 4.1 asks the row's request to match "what the tunnel-management UI sends for a hand-configured
    forward", but that UI only configures SSH tunnels - it has no hand-made Kubernetes forward. The
    row commands and the Tunnels window's new Port forwards section share one request type and one
    app-wide list (`k8s::cluster::port_forwards`); the tests pin that shared request.
  - Services forward to a Running pod their selector picks, on the pod port the `targetPort` names -
    Kubernetes port-forwards to pods only, as `kubectl port-forward svc/...` does.
  - Edit is refused for Secrets: their values are redacted in the object panel, so saving the YAML
    would overwrite the real values with redacted text.
  - The palette now reads `X && !Input`-style command contexts as key bindings do (needed for `?`
    to be offered outside text fields). As a result the object list's, events browser's and
    Resource panel's existing `&& !Input` commands now appear in the palette while their panel has
    focus - they were bound but never offered before.
- Section 10 (#136, #143) is in pilgrimagesoftware/Fernrohr-App#138. Deviations worth a look:
  - The Pods list still offers Delete and Kill without checking the Pod kind's `delete` verb. Every
    other list, and the pod detail panel, gate on it.
  - The object panel doesn't offer Delete while a YAML edit is open, so a delete can't silently
    discard the edit.
  - The refusal banner's Dismiss became an icon button with a tooltip (`icon-buttons.md`). It isn't
    a registered command, since it only clears a message.
