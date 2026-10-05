# Design

## Context

Tunnels today are SSH-only. `config/tunnels.rs` defines `TunnelConfig`, which holds a bastion, a
user, jump hosts and an auth marker. `k8s/cluster/tunnel.rs` builds an `SshTunnelConfig` from that
and acquires a shared forward from the app-scoped `ForwardRegistry`, keyed by
`ForwardKey { tunnel_id, host, port }`. `tunnel/ssh` implements `forward::supervisor::ForwardTransport`
by running `ssh -N -L` in its own process group, and `util/pidfile.rs` records each live `ssh` pid.
On launch and at quit, a sweep kills orphaned or still-live process groups, but only after checking
that the process's command line looks like one of our `ssh -N -L` forwards. `connection/connect.rs`
then points the client at `127.0.0.1:<port>` and pins `tls_server_name`.

The motivating setup (see proposal.md) runs a vendor CLI whose local port is an **HTTP proxy**, not
a direct path to an API server. It also carries a bastion-local port (`8888`) that a user may
already have open in a terminal.

## Goals / Non-Goals

**Goals:**

- Let a command tunnel plug into the existing `ForwardTransport`, supervisor and registry with no
  changes to their contracts.
- Make proxy mode a per-client setting, not a process-wide `HTTPS_PROXY`.
- Leave SSH tunnels' behavior and their `tunnels.toml` representation unchanged.

**Non-Goals:**

- Running the command through a shell. Pipes, `&&`, `$VAR` expansion and `~` expansion are not
  supported.
- Per-API-server placeholders such as `{host}` or `{target_port}`. A command tunnel is one process
  per tunnel.
- SOCKS proxies. kube-rs supports them, but nobody has asked for them yet. The mode enum leaves
  room to add them.
- Attaching to a tunnel the user has already started in a terminal. A fixed port that is already
  in use still fails as "port in use", per the existing `managed-forward` requirement.
- Interactive prompts from the command, such as host-key confirmation or a key passphrase. The
  command has to run unattended. The failure output tells the user to run it once in a terminal
  first.

## Decisions

### 1. `TunnelConfig` gains a tagged kind, flattened for backward compatibility

`TunnelConfig` keeps `name`, and gains `kind: TunnelKind` (`ssh` | `command`, default `ssh`) plus
a `command: CommandTunnelConfig` table (`command_line`, `mode`, `local_port: Option<u16>`,
`startup_timeout_secs`). The SSH fields stay where they are at the top level. That way a file
written before this change deserializes unchanged, and the keychain secrets keyed by tunnel id
don't move.

- **Alternative: a Serde-internally-tagged enum** (`enum TunnelConfig { Ssh {..}, Command {..} }`).
  It is cleaner in Rust, but every existing file would need a migration, and switching a tunnel's
  kind in the editor would throw away its SSH settings. Keeping both lets a user flip the kind and
  flip it back without losing anything.

`no_secret_fields` grows to cover the new fields. The editor shows a hint that the command is
stored in plain text.

### 2. A new `CommandTransport` implements `ForwardTransport`

It lives in `tunnel/command/`, next to `tunnel/ssh/`. It handles:

- **argv.** Strip backslash-newline sequences, then split with the `shlex` crate. `shlex` is
  already in the lockfile as a transitive dependency and gets added as a direct one. Splitting
  happens at save time for validation and again at start. Substitute `{port}` per argument, after
  splitting, so a port can never change how quoting splits the line.
- **spawn.** Use `tokio::process::Command` with `process_group(0)` on unix and `kill_on_drop`.
  stdin is `Stdio::piped()` and the write end is held in the transport, so a command without `-N`
  doesn't see EOF and close its session. stdout and stderr are drained into a bounded ring buffer
  holding the last 50 lines.
- **readiness.** Poll `TcpStream::connect(127.0.0.1:port)` every 250 ms until it succeeds, the
  child exits (failure carries the exit status and the output tail) or `startup_timeout` expires
  (kill the group, failure carries the tail).
- **health.** The supervisor's existing periodic probe becomes "child still running and a TCP
  connect succeeds". The supervisor's backoff loop already handles Reconnecting and keeps the port.
- **stop.** Send `SIGTERM` to the process group, wait up to 3 s, then `SIGKILL`, reusing
  `util::pidfile::kill_process_group`'s mechanism with a TERM variant. Vendor CLIs like `gcloud`
  are wrapper scripts that spawn `ssh`, so a direct-pid kill would leave the real `ssh` running
  while it holds the port. This is the same reason SSH tunnels already use process groups.

- **Alternative: run it through `sh -c`.** That would support pipes and `$VAR`, but it brings
  quoting and injection surprises, and the pid we hold would belong to the shell instead of the
  tunnel. Process-group kill would still catch the descendants. Not having a shell keeps the
  argument the user sees the argument that runs.

### 3. Orphan sweep learns command tunnels

Each command tunnel's pidfile also records its argv[0] basename. Command tunnels write their
pidfiles to a separate `cmd-pids` directory. The startup and quit sweeps for that directory check
that the process's command line starts with the recorded basename, which replaces the
`ssh -N -L` signature check, and keep the same parent-pid checks. That means a crash can't leave a
`gcloud … -L8888` running that would block the next launch's fixed port.

### 4. Login-shell `PATH`

Add a `util::login_env` helper. Once per run, lazily, it runs `$SHELL -lic 'printf %s "$PATH"'`
with a 5 s timeout, delimiting the value so stray output from rc files can be stripped. On failure
it falls back to the process `PATH` plus `/opt/homebrew/bin`, `/usr/local/bin` and `~/.local/bin`.
`CommandTransport` sets that `PATH` on its child. It doesn't modify the app's own environment, so
exec-auth plugins keep behaving as they do today. Widening their `PATH` is a separate decision.

### 5. The forward key varies by tunnel kind

`ForwardKey` becomes an enum: `Ssh { tunnel_id, host, port }` | `Command { tunnel_id }`. The
registry is already generic over the key, so contexts on different API servers bound to one command
tunnel collide on `Command { tunnel_id }` and share one process (see the
`context-tunnel-binding` delta). The registry holds the transports behind a small `TunnelTransport`
enum, or a boxed `ForwardTransport`, whichever fits the supervisor's existing generic bound with
the least churn.

### 6. Proxy mode sets `kube::Config::proxy_url`

`connect.rs` branches on the acquired tunnel's mode:

- **SSH or forward mode:** the existing URL and `tls_server_name` rewrite.
- **Proxy mode:** leave `cluster_url` and `tls_server_name` alone and set
  `config.proxy_url = Some("http://127.0.0.1:<port>")`. kube-rs then sends every request from that
  client through hyper-util's proxy connector with `CONNECT`, so TLS still runs end to end against
  the real host name.

This needs the `kube` crate's `http-proxy` feature. A kubeconfig's own `proxy-url` for that
context, if it has one, is overridden while the tunnel is bound, and a debug log line records it.

- **Alternative: set `HTTPS_PROXY` on the process.** This is what users do with `kubectl` today.
  Rejected because it would send every context and every outbound request (update checks,
  Prometheus `directUrl`, the issue reporter) through one bastion.

Port-forwards and exec go through the same client, so they inherit the proxy. Websocket upgrades
over `CONNECT` are supported by hyper-util's tunnel connector. That is worth confirming with an
exec session during manual verification (task 6.3).

### 7. Editor

The tunnel editor gets a segmented kind switch at the top, and shows the SSH form or the command
form below it. The command form has:

- a multi-line command field (pasted backslash-continued commands keep their shape)
- a mode switch (Proxy / Forward) with one line of help text for each
- an optional local port
- a startup timeout

Validation runs on save, as it does for SSH, and marks fields inline. The tunnel list shows a
small kind badge. Test reuses the existing test flow, using `CommandTransport` with a one-shot
start and stop.

## Risks / Trade-offs

- [A command that prompts, e.g. for a passphrase or host key, hangs until the timeout] → the
  timeout failure shows the output tail, which normally contains the prompt text, and the editor's
  help line says to run the command once in a terminal first.
- [The user's terminal tunnel already holds the fixed port] → it fails as "port in use", naming the
  port. This is clear and recoverable.
- [The login shell is slow or noisy (`-i` sources rc files)] → the timeout plus fallback `PATH`,
  the result is cached for the run, and delimited capture.
- [The proxy only allows `CONNECT` to some ports] → the failure is surfaced through the
  `Proxy refuses the connection` scenario. There's no silent fallback.
- [The command line holds a credential] → it is stored in plain text in `tunnels.toml`. The editor
  says so, and documentation recommends getting credentials from the CLI's own auth store, not
  from flags.
- [Windows] → process groups and the login-shell `PATH` are unix-only. On other platforms,
  `CommandTransport` uses `kill_on_drop` without the group, the same as `ssh` there today.

## Migration Plan

None needed. A missing `kind` deserializes as `ssh`. Rolling back to an older build loads command
tunnels as SSH tunnels with an empty host. They fail validation, and the user can delete them. The
next save by the old build drops the `kind` and `command` fields, which is acceptable for a
downgrade.
