## Why

Real clusters often sit behind a bastion: the API server is only reachable through an SSH tunnel. A
user working several environments at once (for example two QA clusters sharing one QA bastion, one
prod cluster on a separate bastion, and a local cluster needing none) needs Fernrohr to define those
tunnels once and bind each kube context to the right one. The same machinery - a long-lived, managed,
health-checked local forward - is also what Kubernetes `port-forward` needs, so the two are built as
one subsystem. This change depends on `bootstrap-fernrohr`.

## What Changes

- New `ManagedForward` subsystem: a long-lived local forward with a connection state machine
  (Disconnected → Connecting → Up → Reconnecting), health checking, automatic local-port allocation,
  reference counting, and a status row in the UI. Two implementations:
  - `SshTunnel`: forwards `127.0.0.1:PORT` to a remote host/port through an SSH bastion, with support
    for a jump-host chain. Implemented by shelling out to `ssh -L` in this change; the trait boundary
    lets a native (`russh`) implementation replace it later without touching callers.
  - `K8sPortForward`: forwards a local port to a Pod/Service port via the Kubernetes port-forward API.
- Tunnel definitions: create, edit, and delete named SSH tunnel configs (host, port, user, auth
  method, jump hosts, keepalive). Non-secret fields persist to `tunnels.toml`; SSH key passphrases and
  passwords go to the OS keychain via the `keyring` crate, never to disk.
- Context binding: bind a kube context to zero or one tunnel; many contexts may share a tunnel. A
  bound context connects only once its tunnel is `Up`. Binding is persisted.
- Connection rewrite: when a context is bound, the Kubernetes client is built against
  `https://127.0.0.1:<allocated-port>` with the TLS server name pinned to the real API server host so
  certificate validation still passes.
- Watcher resilience: a tunnel dropping pauses that cluster's watch streams instead of tearing them
  down; when the tunnel returns to `Up`, the watches resume and converge. The same pause/resume path
  covers exec credential-plugin re-authentication (for example `aws eks get-token`) after a 401.
- Tunnel lifecycle: reference-counted - a tunnel starts when a bound, active cluster session needs it
  and stops when the last such session releases it.
- Keychain fallback on Linux: when no Secret Service daemon is available, prompt for the secret each
  session rather than failing.

Non-goals: the native `russh` implementation, `K8sPortForward` UI as a full "port-forward manager"
view (the implementation lands here; the dedicated management view is a later change), Prometheus
`directUrl` wiring (a later metrics change consumes `ManagedForward`), and SOCKS/dynamic forwarding.

## Capabilities

### New Capabilities

- `managed-forward`: the shared local-forward abstraction - state machine, health checks, local-port
  allocation, reference counting, and observable status - with SSH-tunnel and Kubernetes
  port-forward implementations.
- `tunnel-config`: defining, editing, deleting, and persisting named SSH tunnel configurations, with
  secrets stored in the OS keychain.
- `context-tunnel-binding`: binding a kube context to a tunnel, gating connection on tunnel
  readiness, and rewriting the client URL and TLS server name accordingly.

### Modified Capabilities

- `cluster-connection`: connecting a context now waits for a bound tunnel to be `Up` before building
  the client, builds it against the tunnel's local address with a pinned TLS server name, and treats
  a tunnel drop or a credential-plugin 401 as a recoverable pause of that connection's watches rather
  than a disconnect.

## Impact

- New dependency: `keyring`. `ssh` must be present on `PATH` (documented as a requirement); child
  `ssh` processes are spawned and supervised.
- New modules: `managed_forward` (trait, state machine, port allocator, registry), `ssh_tunnel`,
  `k8s_port_forward`, tunnel-config store + keychain wrapper, tunnel/binding UI, connection-rewrite
  logic in the cluster session.
- Modified: `cluster-connection` connect path and the watch registry from `bootstrap-fernrohr` gain a
  paused state; `paths.rs` gains `tunnels.toml`.
- Security: secrets are keychain-only; `tunnels.toml` holds no credentials. TLS validation is
  preserved through the tunnel via server-name pinning - no `insecure-skip-tls-verify`.
- Risk: `ssh -L` readiness and failure signaling is coarse (poll the local port, scrape stderr);
  known-hosts prompts and passphrase prompts are delegated to `ssh` itself for now.
