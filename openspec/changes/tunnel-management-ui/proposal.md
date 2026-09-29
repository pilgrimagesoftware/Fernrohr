# Proposal

## Why

`tunnel-subsystem` shipped the tunnel store and connect path but no UI, so today tunnels and
context bindings exist only as a hand-edited `tunnels.toml`. Verifying against the real QA IAP
bastions (`tunnel-bastion-verification`) also exposed a model mismatch: each tunnel carries its own
`remote_host`/`remote_port`, so one bastion that reaches every cluster needs a duplicate tunnel entry
per cluster, and nothing checks that target against the context's actual API server.

## What Changes

- A **Tunnels panel** to create, edit, rename, test, and delete tunnels. A tunnel describes only how
  to reach a bastion: host (a hostname or `~/.ssh/config` alias), port, user, jump hosts, and
  authentication (ssh config/agent, or a private key held in the OS keychain).
- **BREAKING (`tunnels.toml`)**: tunnels no longer carry `remote_host`/`remote_port`. The forward
  target is derived at connection start from the connecting context's kubeconfig `server:` URL.
  Existing files load with those fields ignored and are rewritten without them on the next save.
- **Binding is set from the context side.** Each context in the cluster picker gets a tunnel
  selector ("Direct" or one tunnel), and the command palette gets "Set tunnel for context". The
  tunnel a context uses is resolved when its connection starts; changing it never touches a live
  connection. The Tunnels panel does not assign contexts. It shows only a read-only count of the
  contexts that use each tunnel, and on delete it names the contexts that fall back to Direct.
- Forwards are shared per **(tunnel, API server host:port)**. Two contexts that point at the same
  cluster through the same tunnel share one `ssh` process. Different clusters through one bastion
  each get their own forward.
- Bindings whose context no longer exists in the kubeconfig (for example after a
  `kubectl config rename-context`) are flagged in the Tunnels panel and can be cleared there,
  instead of silently disappearing.

Not in scope: moving paused/reconnecting state into a window status bar (a separate change),
multiplexing several forwards over one `ssh` connection (`ControlMaster`), and a built-in IAP
transport. IAP bastions work through a `ProxyCommand` in `~/.ssh/config`, as verified.

## Capabilities

### New Capabilities

None. The behavior lives in the existing tunnel capabilities.

### Modified Capabilities

- `tunnel-config`: tunnels drop their forward target; tunnel management gains a panel, a
  connection test, and read-only usage counts; delete falls back to Direct.
- `context-tunnel-binding`: bindings are chosen per context and resolved at connection start; the
  forward target comes from the context's API server; forwards are shared per (tunnel, target);
  stale bindings are surfaced.

## Impact

- App code: `config/tunnels.rs` (schema), `tunnel/store.rs` (legacy-field load, usage and stale
  queries), `k8s/cluster/tunnel.rs` (target derivation, registry key), `forward/registry.rs` (key
  type), `ui/picker.rs` (binding selector, kept in its own module because `picker.rs` is already
  close to the 700-line limit), and a new Tunnels panel under `ui/panel/`, plus new commands in the
  command registry.
- Config: `tunnels.toml` loses two fields per tunnel. Old files keep loading.
- No new dependencies.
