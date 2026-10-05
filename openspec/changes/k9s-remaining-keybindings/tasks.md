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

- [ ] 7.1 Add a `global.show_key_hints` command (default binding `?`) that opens an overlay built from
      the `CommandRegistry` filtered to the currently focused `KeyContext`, and verify a test confirms
      the listed commands match what the palette would show in that same context
- [ ] 7.2 Show commands with no current binding as unbound rather than omitting them, and verify a
      test covers a command whose binding was removed via the keybindings editor
- [ ] 7.3 Verify dismissing the overlay returns keyboard focus to its prior location via a focus-order
      test

## 8. Command registry and keymap wiring

- [ ] 8.1 Register all new command ids and default bindings (`pods.delete`, `pods.kill`,
      `object_detail.edit`, `pods.shell`, `pods.port_forward`, resource-panel quick-jump commands,
      `global.show_key_hints`) and verify `keymap.toml`'s first-run defaults include every new id
- [ ] 8.2 Add palette and keybindings-editor coverage for each new command by relying on the existing
      registry-driven rendering, and verify a test confirms each new command id appears in both
      without additional per-command UI code
