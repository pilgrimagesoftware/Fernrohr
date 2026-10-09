# Tasks

## 1. Manual kind configuration

- [x] 1.1 Add `TunnelKind::Manual` and the flattened `manual` settings (`message`, `skip_when_reachable` defaulting to true) to `TunnelConfig`; verify `tunnels.toml` round-trip tests, including switching kind without losing other kinds' settings and loading files with no `manual` table.
- [x] 1.2 Add the manual kind to the tunnel editor (message field, skip-when-reachable toggle) and a manual badge in the tunnel list; verify keyboard-driven editor tests (`simulate_keystrokes`) create and switch a tunnel to the manual kind.

## 2. Confirmation lifecycle

- [x] 2.1 Add a non-retryable outcome to `ForwardTransport::connect` that makes the supervisor close the forward's state channel with the transport's reason; verify SSH and command transports keep their retry behavior and `connect_and_probe` fails with that reason.
- [x] 2.2 Add `ForwardKey::Manual { tunnel_id }`, `ManualTransport`, and the `ManualConfirmations` global with Proceed and Cancel resolution, entry removal on forward drop, and no client address rewrite; verify tests for proceed, cancel, two contexts sharing one entry, an already-confirmed tunnel connecting without a prompt, a re-prompt after the last disconnect, and a re-prompt after a failed connection following Proceed.
- [x] 2.3 Implement the skip-when-reachable TCP probe with a 2-second timeout; verify tests for reachable, unreachable, and setting-off cases using a local listener.
- [x] 2.4 Verify non-blocking behavior: with one context awaiting confirmation, a test connects a second, unbound context and loads its data while the first still waits.

## 3. Desktop notifications

- [x] 3.1 Add the `notify` module backed by `notify-rust`, posting on a blocking task, logging failures, and reporting activation where supported; verify unit tests through a fake backend for success, failure-is-logged-only, and activation focusing the originating window.

## 4. Prompt UI and commands

- [ ] 4.1 Add `ContextHealth::AwaitingConfirmation` and `Severity::Attention`, and draw the waiting context's capsule filled in the attention color with a distinct awaiting-confirmation state icon whose tooltip gives "Awaiting confirmation", the elapsed wait, and the tunnel's message, sorted ahead of every other state; verify render tests for the fill, text, tooltip, and sort order.
- [ ] 4.2 Add Proceed and Cancel, with their bound keys, above Disconnect in the waiting capsule's menu; verify keyboard tests open the menu with real keystrokes and that Proceed or Cancel from either of two capsules sharing a tunnel resolves both, while Disconnect closes only its own context.
- [ ] 4.3 Show the awaiting-confirmation line with Proceed and Cancel in the cluster picker's status line; verify picker tests for the waiting, proceed, and cancelled states.
- [ ] 4.4 Register `tunnel.manual.proceed` and `tunnel.manual.cancel` with default bindings and a pending-tunnel picker when several are waiting; verify keystroke tests for the single-entry and multi-entry cases.
- [ ] 4.5 Post one desktop notification when a pending entry is created, naming the tunnel and showing its message; verify that a second context joining the entry posts nothing.
- [ ] 4.6 Document manual tunnels (purpose, settings, prompt surfaces, Proceed and Cancel keys, macOS development-build notification caveat) in the user docs; verify the docs contain no infrastructure identifiers.

## 5. Integration

- [ ] 5.1 Add an end-to-end test with recorded fixtures covering two contexts on one manual tunnel, an unrelated context connecting during the wait, Proceed, and Cancel; verify `cargo fmt -- --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` pass.
