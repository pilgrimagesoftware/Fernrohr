# Design

## Context

See proposal.md for motivation. Current state in the App repo:

- `TunnelStore` (`tunnel/store.rs`) already has `create`/`update`/`delete`/`bind`/`unbind`/
  `bindings`/`binding_for`/`secret`, and `delete` returns the contexts it unbound. No UI calls it.
- `TunnelConfig` (`config/tunnels.rs`) has `remote_host`/`remote_port`. Its serde derive does not set
  `deny_unknown_fields`.
- `ClusterConnection::connect` (`k8s/cluster/connection.rs`) resolves the binding and acquires the
  forward **synchronously on the GPUI foreground thread** (acquiring needs `&mut App`), before the
  async `resolve_config` runs. So the forward target must be known without the async config.
- `ForwardRegistry` (`forward/registry.rs`) is keyed by `String` (today the tunnel id) over
  `Weak<F>` entries.
- The cluster picker (`ui/picker.rs`, 598 lines) is the first view of each window. It already
  renders `WaitingForTunnel`.
- Verified on 2026-09-29: IAP bastions work unchanged when `bastion_host` is a `~/.ssh/config`
  alias whose `ProxyCommand` runs `gcloud compute start-iap-tunnel`.

## Goals / Non-Goals

**Goals:**
- Manage tunnels and bindings without editing TOML, from the context side for bindings.
- Make one tunnel definition serve every cluster behind a bastion.

**Non-Goals:**
- Multiplexing several forwards over one SSH connection (`ControlMaster`). Each (tunnel, target)
  forward is its own `ssh` process, matching today's supervisor model.
- Editing `~/.ssh/config` from the app. The editor accepts an alias and points users to their SSH
  config for `ProxyCommand` setups.
- Password authentication. The app never passes a password to `ssh` (it would need a TTY or an askpass
  helper). Auth stays "ssh config/agent" or "private key in keychain".

## Decisions

### 1. Forward target comes from a synchronous kubeconfig read

Add `kubeconfig::server_for_context(path, context) -> Result<(host, port)>`. It uses the same
`Kubeconfig::read` the picker already uses, follows context → cluster → `server`, and defaults the port
from the scheme (443 for `https`). `acquire_for_context` calls it before `ForwardRegistry::acquire`.

*Alternative:* acquire after the async `resolve_config` produces `Config.cluster_url`. That means
hopping back to the foreground for `&mut App` mid-connect and splitting the `WaitingForTunnel` state
across two tasks. The sync read is cheap, and `Kubeconfig::read` is already sync.

*Consistency:* `rewrite_for_tunnel` still reads the host from the resolved `Config.cluster_url` to pin
`tls_server_name`. Both come from the same kubeconfig entry. A test covers a context whose server has
an explicit port and one that doesn't.

### 2. Registry key becomes a typed `ForwardKey { tunnel_id, host, port }`

Make `ForwardRegistry` generic over `K: Hash + Eq + Clone` (the SSH path uses `ForwardKey`, and any
k8s port-forward callers keep their current key). Contexts pointing at the same API server through the
same tunnel collide on the key and share one `ssh`. Different API servers get separate entries.

*Alternative:* a composite string `"{id}|{host}:{port}"`. It's rejected because a typed key is what
lets the running-state query group by `tunnel_id` without parsing strings.

### 3. Registry publishes its live key set

`ForwardRegistry` gains a `watch::Sender<BTreeSet<K>>`, updated on first acquire and on last release.
The Tunnels window subscribes through the existing `runtime::spawn_stream`/`drain` bridge and derives
"running (n forwards)" per tunnel id. *Alternative:* poll on a timer. It's rejected: laggy, and it
wakes the UI for nothing.

### 4. Schema: drop the target fields, let serde ignore them on load

Remove `remote_host`/`remote_port` from `TunnelConfig`. With no `deny_unknown_fields`, old files
parse, and the next `TunnelStore` write omits them. Add `auth: TunnelAuth` (`SshConfig` default |
`KeychainKey`). This stays a non-secret marker for whether a keychain entry is expected, so the
`no_secret_fields` test keeps holding. `jump_hosts` and keepalive stay as they are.

### 5. Tunnels UI is its own single-instance window

Tunnels are app-global, not per-cluster, and must be reachable before any context is connected, so
the Tunnels UI can't be a cluster dock panel. It opens as one OS window, reusing
`util/shell.rs`'s window setup. It's reached from a "Manage tunnels…" control in the picker, the app
menu, and a `tunnels.manage` command. It has a list (name, host, usage count, running state, stale
bindings) and an editor pane (fields, auth, jump hosts, Test, Save, Delete with a confirm that lists
the contexts falling back to Direct). Files: `ui/tunnels/{mod,list,editor}.rs`, each under the
700-line cap.

### 6. Binding selector lives on each picker row, in its own module

`ui/picker_tunnel.rs` renders a compact dropdown (Direct / each tunnel by name) on each context row
and writes through `TunnelStore::bind`/`unbind`. A `context.set_tunnel` palette command opens the
same list for the focused or current context. Because `connect` re-reads `tunnels.toml` on every
connect (unchanged), "resolved at connection start, never alters a live connection" needs no extra
code. The live connection simply holds its existing `RegistryHandle`.

### 7. Test runs `ssh` without a forward

The Test button spawns the same argv as a real tunnel minus `-N -L`, plus `-o BatchMode=yes -o
ConnectTimeout=15`, and runs the remote command `true`. It reports exit 0, or the stderr tail on
failure. It runs on the tokio runtime and never touches `ForwardRegistry`.

### 8. Stale bindings

On picker load (which already lists contexts), compare `TunnelStore::bindings()` keys against the
context names. The Tunnels window shows the difference with a per-row "Remove" that calls `unbind`.

## Risks / Trade-offs

- [One `ssh` + IAP session per (tunnel, cluster)] → Fine at today's scale. `ControlMaster` is the
  listed follow-up if bastion session counts become a problem.
- [Test's `true` command may be blocked on bastions with a `ForceCommand`/no-shell policy] → Report
  the failure verbatim. A real tunnel (`-N`) may still work there. The editor notes that the test
  runs a command.
- [Sync kubeconfig read on the foreground thread] → The same cost the picker already pays, and only
  once per connect.
- [Readiness timeout of 5 s vs ~4 s IAP connect seen in verification] → Out of scope here. It's
  recorded in `tunnel-bastion-verification` findings.

## Migration Plan

No user action. Existing `tunnels.toml` files load with the target fields ignored, and their
bindings keep working because the target now comes from the kubeconfig. Rollback: an older build
would reject a rewritten file that lacks `remote_host`/`remote_port`. That's acceptable pre-1.0, and
it's noted in the release notes.
