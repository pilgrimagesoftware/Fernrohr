## 1. Prerequisites and spike

- [ ] 1.1 Add the `keyring` dependency; a smoke test writes and reads back a secret on macOS Keychain and, where available, Linux Secret Service, and reports a catchable error when no daemon is present
- [ ] 1.2 Confirm the TLS rewrite: build a `kube` client with `cluster_url` set to a local port and `tls_server_name` set to a different host, and verify against a bastioned test cluster (or a local stand-in with a SAN mismatch) that the certificate validates against the pinned name - this gates section 5
- [ ] 1.3 Document `ssh` on `PATH` as a runtime requirement in the README and fail fast with a clear message if it is missing

## 2. ManagedForward core

- [ ] 2.1 Define `ForwardState` (Disconnected/Connecting/Up/Reconnecting) and the `ManagedForward` trait (`state()` receiver, `local_addr()`, handle-based acquire/release); unit tests drive a fake implementation through every transition
- [ ] 2.2 Implement the local-port allocator (bind `127.0.0.1:0`, hand back the port; honor an explicit port and report conflicts); tests cover auto-allocation and explicit-port-in-use
- [ ] 2.3 Implement the per-forward supervisor task: owns the state machine, runs a health-check tick, and applies backoff on failure; tests with a fake transport cover transient-drop recovery keeping the same port and sustained-failure staying in Reconnecting
- [ ] 2.4 Implement `ForwardRegistry` (app-scoped) with identity keying and reference counting; tests assert one underlying forward for two acquirers and teardown on last release

## 3. SSH tunnel implementation

- [ ] 3.1 Implement `SshTunnel`: spawn and supervise `ssh -N -L ... [-J ...] user@bastion` with `ExitOnForwardFailure`, `ServerAliveInterval`, and `BatchMode` where a secret is available; a test against a local sshd container brings a forward Up and proxies a TCP echo
- [ ] 3.2 Implement readiness by local-port probe and failure by child exit / failed probe; tests cover auth failure and forward-bind failure each producing a distinct reason string
- [ ] 3.3 Implement jump-host chains (`-J`); a test through two chained local sshd containers establishes the forward in order
- [ ] 3.4 Implement child-process cleanup: process-group kill on drop and a startup sweep for orphaned forwards from a stale pid file; a test kills the supervisor and asserts no `ssh` child survives

## 4. Kubernetes port-forward implementation

- [ ] 4.1 Implement `K8sPortForward`: local `TcpListener` on the allocated port, each connection bridged to a fresh `kube` port-forward stream; a test against a mock/kind cluster forwards to a Service port and serves a request
- [ ] 4.2 Handle target loss: deleting the backing Pod moves the forward to Reconnecting and it recovers when a matching target returns; covered against a kind cluster or a faithful mock
- [ ] 4.3 Confirm no user-facing port-forward management surface was added in this change (implementation only)

## 5. Tunnel configuration and secrets

- [ ] 5.1 Define the `tunnels.toml` schema (tunnel configs + `context -> tunnel id` bindings) on the typed-TOML helper; serde round-trip tests, and a test asserting no secret fields exist in the struct
- [ ] 5.2 Implement the keychain wrapper (service = app id, account = tunnel id) with a per-session in-memory prompt fallback when the keychain is unavailable; tests cover save/read/delete and the fallback path
- [ ] 5.3 Implement tunnel CRUD UI (create, edit, rename, delete) writing non-secret fields to `tunnels.toml` and secrets to the keychain; deleting a tunnel warns which contexts unbind and removes the keychain entry - manual check plus a store-level test for the unbind cascade

## 6. Context binding and connect-path integration

- [ ] 6.1 Implement binding UI and persistence: bind a context to zero or one tunnel, many contexts to one tunnel, persisted in `tunnels.toml`; relaunch keeps bindings - store test plus manual check
- [ ] 6.2 Extend the connect path: for a bound context, acquire the forward from `ForwardRegistry`, show a waiting-for-tunnel state, await Up, then rewrite `cluster_url` + `tls_server_name` before the existing client build; release the forward when the last session using it disconnects - tested against a local sshd + mock API server for wait, success, and tunnel-fails-to-start
- [ ] 6.3 Verify an unbound context still connects directly with no `ForwardRegistry` involvement (regression against `bootstrap-fernrohr` behavior)

## 7. Recoverable interruptions

- [ ] 7.1 Add a `Paused` state to the bootstrap `WatchRegistry` entries with suspend/resume calls; unit tests assert a paused watcher stops consuming and, on resume, relists and reconverges
- [ ] 7.2 Implement `ConnectionHealth` per `ClusterSession` driven by the bound forward's state receiver; a forward transition Up→Reconnecting pauses that cluster's watchers and Reconnecting→Up resumes them - tested with a fake forward and mock streams
- [ ] 7.3 Wire exec-plugin `401` handling into the same path: a `401` that a credential refresh resolves resumes watches without closing panels; an unrecoverable sustained failure plus user disconnect releases them - tested against a mock API server that returns `401` then `200`
- [ ] 7.4 Surface the paused state and its reason/elapsed time on affected panels - manual check

## 8. Integration verification

- [ ] 8.1 End to end on macOS and Linux: define two tunnels, bind three contexts (two sharing one tunnel, one on the other, one unbound), connect all four, confirm two underlying `ssh` forwards and one direct connection, and confirm `tunnels.toml` holds no secrets
- [ ] 8.2 Flap test: kill one bastion's `ssh` forward mid-session, confirm the two dependent connections pause and their panels stay open, restore the bastion, confirm both resume and their Pods tables reconverge
- [ ] 8.3 Credential test against a cluster using an exec plugin (or a faithful stand-in): force a token expiry, confirm re-auth resumes watches with no panel loss
- [ ] 8.4 Confirm TLS: connect a bastioned context and assert the connection validates against the real API server host with no insecure flag anywhere in the path
