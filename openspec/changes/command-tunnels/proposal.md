# Proposal

## Why

Fernrohr's only tunnel is a plain SSH tunnel to a bastion, so it can't use clusters that are reached
another way. A common setup runs a vendor CLI that starts an SSH session through an identity-aware
proxy, such as
`gcloud compute ssh <host> --tunnel-through-iap -- -L8888:127.0.0.1:8888`. That session forwards a
local port to an HTTP proxy on the bastion, and `kubectl`, `k9s` or `flux` then run with
`HTTPS_PROXY=http://localhost:8888`. Today the user has to start that command in another terminal
and keep it running, and Fernrohr still has no way to send its traffic through the proxy.

## What Changes

- Add a second tunnel kind, **command tunnel**. Fernrohr starts and supervises a user-supplied
  command line instead of building its own `ssh -L` invocation. A `{port}` placeholder in the
  command is replaced with the local port Fernrohr chose. A command with a fixed port instead names
  that port in the tunnel's settings.
- Each command tunnel has a **mode** that says what its local port offers:
  - `proxy` (the default) is an HTTP proxy. A context bound to it connects to its real API server
    URL through the proxy, so no address rewrite is needed.
  - `forward` reaches the API server directly. A context bound to it uses the existing loopback
    address and TLS server name rewrite.
- A command tunnel counts as Up when its process is running and its local port accepts
  connections. If the process exits or the port stops answering, the tunnel reconnects with the
  existing backoff. Stopping the tunnel ends the command's whole process group.
- The Tunnels panel editor gains a kind selector and the command-tunnel fields: command, mode,
  optional fixed local port and startup timeout. Testing a command tunnel runs the command until it
  is ready, then stops it, and shows the command's output if that fails.
- Every context bound to a command tunnel shares one running command, whatever its API server.
- Existing SSH tunnels and `tunnels.toml` files keep working unchanged. **BREAKING**: none.

## Capabilities

### New Capabilities

(none)

### Modified Capabilities

- `tunnel-config`: a tunnel is either an SSH tunnel or a command tunnel. Command tunnels add
  command, mode, local-port and timeout fields, their own validation rules, and a test that runs
  the command until it is ready.
- `managed-forward`: adds a command-backed implementation. It spawns and supervises a user command,
  decides readiness by probing the local port, and stops the process group on release.
- `context-tunnel-binding`: proxy-mode tunnels route the client through an HTTP proxy instead of
  rewriting its address. The address and TLS rewrite applies only to SSH and forward-mode tunnels.
  Command tunnels are shared per tunnel, not per tunnel and API server.

## Impact

- `App/app/src/config/tunnels.rs`: `TunnelConfig` gains a kind and command fields. A file without
  a kind loads as SSH, and the `no_secret_fields` guard still holds.
- `App/app/src/tunnel/`: a new `command` module beside `ssh`, covering argv parsing, placeholder
  substitution, spawning in its own process group, the readiness probe and the output tail.
- `App/app/src/k8s/cluster/tunnel.rs` and `connection/connect.rs`: per-kind forward keys, and
  setting `kube::Config::proxy_url` for proxy mode.
- `App/app/src/ui/tunnels/editor*`: a kind selector and the command-tunnel form.
- `App/app/Cargo.toml`: `kube` needs the `http-proxy` feature (hyper-util's client proxy).
- The app has to resolve the user's login-shell `PATH` so a CLI like `gcloud` is found when
  Fernrohr is launched from the Dock or a desktop launcher instead of a terminal.
