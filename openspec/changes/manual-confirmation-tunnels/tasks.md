# Tasks

## 1. Manual kind configuration

- [ ] 1.1 Add `TunnelKind::Manual` and the flattened `manual` settings (`message`, `skip_when_reachable` defaulting to true) to `TunnelConfig`; verify `tunnels.toml` round-trip tests, including switching kind without losing other kinds' settings and loading files with no `manual` table.
- [ ] 1.2 Add the manual kind to the tunnel editor (message field, skip-when-reachable toggle) and a manual badge in the tunnel list; verify keyboard-driven editor tests (`simulate_keystrokes`) create and switch a tunnel to the manual kind.

## 2. Confirmation lifecycle

- [ ] 2.1 Add a non-retryable outcome to `ForwardTransport::connect` that makes the supervisor close the forward's state channel with the transport's reason; verify SSH and command transports keep their retry behavior and `connect_and_probe` fails with that reason.
- [ ] 2.2 Add `ForwardKey::Manual { tunnel_id }`, `ManualTransport`, and the `ManualConfirmations` global with Proceed and Cancel resolution, entry removal on forward drop, and no client address rewrite; verify tests for proceed, cancel, two contexts sharing one entry, an already-confirmed tunnel connecting without a prompt, a re-prompt after the last disconnect, and a re-prompt after a failed connection following Proceed.
- [ ] 2.3 Implement the skip-when-reachable TCP probe with a 2-second timeout; verify tests for reachable, unreachable, and setting-off cases using a local listener.
- [ ] 2.4 Verify non-blocking behavior: with one context awaiting confirmation, a test connects a second, unbound context and loads its data while the first still waits.

## 3. Desktop notifications

- [ ] 3.1 Add the `notify` module backed by `notify-rust`, posting on a blocking task, logging failures, and reporting activation where supported; verify unit tests through a fake backend for success, failure-is-logged-only, and activation focusing the originating window.

## 4. Prompt UI and commands

- [ ] 4.1 Add `ContextHealth::AwaitingConfirmation` and its status-bar item showing the tunnel name, message, and elapsed time, with Proceed and Cancel icon buttons and tooltips that show their keys; verify render tests find the buttons by id and assert the tooltip text.
- [ ] 4.2 Show the awaiting-confirmation line with Proceed and Cancel in the cluster picker's status line; verify picker tests for the waiting, proceed, and cancelled states.
- [ ] 4.3 Register `tunnel.manual.proceed` and `tunnel.manual.cancel` with default bindings and a pending-tunnel picker when several are waiting; verify keystroke tests for the single-entry and multi-entry cases.
- [ ] 4.4 Post one desktop notification when a pending entry is created, naming the tunnel and showing its message; verify that a second context joining the entry posts nothing.
- [ ] 4.5 Document manual tunnels (purpose, settings, prompt surfaces, Proceed and Cancel keys, macOS development-build notification caveat) in the user docs; verify the docs contain no infrastructure identifiers.

## 5. Integration

- [ ] 5.1 Add an end-to-end test with recorded fixtures covering two contexts on one manual tunnel, an unrelated context connecting during the wait, Proceed, and Cancel; verify `cargo fmt -- --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` pass.
