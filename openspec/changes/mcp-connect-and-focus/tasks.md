# Tasks

## 1. Bring to front

- [ ] 1.1 Add a shared `focus_window` helper (`cx.activate(true)` plus `window.activate_window()`) and call it from `open_panel` and `load_layout` on the window they acted on; verify with a GPUI test that both activate their window, and that `list_layouts` and a read tool leave the active window unchanged.

## 2. Connect context tool

- [ ] 2.1 Add a connect approval request to the approval gate (recoverable tier, naming the context, bound tunnel, and tunnel kind); verify keyboard tests (`simulate_keystrokes`) for approve, deny, and timeout, and that denial starts no connection.
- [ ] 2.2 Add `connect_context` (Navigate kind): kubeconfig resolution, an already-held context focused without approval, `add_context` in the frontmost workspace window or a new window when none, focus, and the 20-second settle wait with the state result; verify tests for unknown, already held, approved connected, failed with a redacted reason, awaiting confirmation through a manual tunnel, and a timeout reporting `connecting`.
- [ ] 2.3 Update the README's Agents (MCP) section and the in-app Agent Access description for the new tool and the bring-to-front behavior; verify the docs contain no infrastructure identifiers.

## 3. Integration

- [ ] 3.1 Extend the MCP client integration tests: the tool list includes `connect_context`, an approved connect through the real adapter reaches connected against a fixture API server, and a denied connect starts nothing; verify `cargo fmt -- --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` pass.
