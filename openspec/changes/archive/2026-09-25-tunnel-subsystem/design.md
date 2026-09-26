## Context

Builds on `bootstrap-fernrohr`: the `tokio` runtime spine, the `ClusterSession` + reference-counted
`WatchRegistry`, and the `paths.rs` / typed-TOML persistence layer already exist. See proposal.md -
Why. The new work is a supervised local-forward subsystem and the changes to the connect path that
route a bound context through it while keeping TLS validation intact.

## Goals / Non-Goals

**Goals:**

- One `ManagedForward` abstraction that SSH tunnels and Kubernetes port-forwards both satisfy, so the
  later port-forward manager and the metrics `directUrl` path reuse it unchanged.
- A connect path where "bound context" is the only new branch: acquire forward, await Up, rewrite
  URL + TLS server name, then the existing client build runs unchanged.
- Tunnel flap and credential-plugin `401` share one "pause the watches, resume on recovery" path.
- Secrets never touch disk.

**Non-Goals:**

- Native SSH (`russh`). The trait boundary is real now but the only implementation shells out to
  `ssh`.
- A port-forward manager view. `K8sPortForward` is implemented and testable but its only caller in
  this change is internal.
- Prometheus wiring. The metrics change consumes `ManagedForward`; nothing here imports it.
- SOCKS / dynamic (`-D`) forwarding.

## Decisions

### D1: `ManagedForward` trait + a `ForwardRegistry` that owns supervision

`trait ManagedForward` exposes `state() -> watch::Receiver<ForwardState>`, `local_addr()`, and
`acquire()`/`release()` semantics via a handle. A `ForwardRegistry` (app-scoped, sibling of
`ClusterRegistry`) keys forwards by identity (`tunnel id` or `cluster + target`), reference counts
them, and runs one supervisor task per live forward on the `tokio` runtime. The supervisor owns the
state machine, health checks, and backoff.

- _Why a registry rather than each `ClusterSession` owning its tunnel?_ Many contexts share one
  tunnel; the refcount and the single supervisor must live above the session. Mirrors the
  `WatchRegistry` pattern from bootstrap.
- _Alternative considered:_ model a tunnel as just another watched resource - rejected, the lifecycle
  and failure semantics are different enough that reuse would be forced.

### D2: `SshTunnel` shells out to `ssh -L`, readiness by port probe

The supervisor spawns `ssh -N -L 127.0.0.1:<port>:<remote_host>:<remote_port> [-J jump,...] <user>@<bastion>`
with `-o ExitOnForwardFailure=yes -o ServerAliveInterval=... -o BatchMode=...` and supervises the
child. `Up` is declared when a TCP connect to the local port succeeds; failure is detected by child
exit or a failed probe. Passphrase and known-hosts interaction are left to `ssh` (askpass / the
user's `~/.ssh/known_hosts`).

- _Why shell out?_ `ssh -L` gets jump-host chains, `~/.ssh/config`, agent auth, and host-key
  handling for free. `russh` means reimplementing all of it. The trait lets us swap later.
- _Trade-off:_ coarse status. We get "up or not" and stderr text, not per-channel events. Acceptable
  for now; documented as the reason the native impl exists on the roadmap.
- _`BatchMode`:_ used when a keychain secret is available (feed via an askpass shim); interactive
  prompt path is the fallback.

### D3: `K8sPortForward` via the kube port-forward API

Implemented with `kube`'s portforward support: a local `TcpListener` on the allocated port, each
accepted connection bridged to a fresh port-forward stream to the target Pod (Services resolved to a
current backing Pod at connect time). Same supervisor, state machine, and refcount as `SshTunnel`.

- _Why include it in this change?_ It is the same abstraction; building both now proves the trait is
  not SSH-shaped. Its UI is deferred, not its code.

### D4: Connection rewrite via `kube::Config` fields, TLS server name pinned

For a bound context the connect path loads the context's `kube::Config`, then before
`Client::try_from` sets `cluster_url = https://127.0.0.1:<port>` and `tls_server_name =
<original host>`. `rustls` then presents SNI and validates the cert chain against the real host while
the socket goes to loopback. No CA changes, no `accept_invalid_certs`.

- _Verified assumption:_ `kube::Config` exposes `tls_server_name`; a task confirms it end to end
  against a bastioned cluster before the rest of the connect work.
- _Alternative considered:_ a local TLS-terminating proxy - rejected, far more moving parts and it
  would need its own cert trust.

### D5: One pause/resume path for tunnel flap and credential `401`

The `WatchRegistry` from bootstrap gains a `Paused` state per `(cluster, kind)` entry. A
`ConnectionHealth` signal per `ClusterSession` is driven by: (a) the bound forward's state receiver -
`Up` → healthy, anything else → paused; (b) the client's auth layer reporting a `401` that triggers
exec-plugin refresh - paused until the retried request succeeds. On `healthy` the registry resumes
each paused watcher, which relists and reconverges (the drain + coalescing from bootstrap absorbs the
relist burst).

- _Why unify?_ Both are "transport briefly unusable, don't lose the user's panels." Separate handling
  would duplicate the suspend/resume plumbing.
- _Bound on paused time:_ none by design - it stays paused until recovery or an explicit user
  disconnect. A visible paused state on the panel makes the condition legible.

### D6: Persistence and secrets

`tunnels.toml` (state dir) holds an array of tunnel configs (id, name, host, port, user, auth
*method*, jump hosts, keepalive) and the `context -> tunnel id` binding map. Secrets are stored via
`keyring` under a service name of the app id and an account of the tunnel id. On a platform with no
Secret Service, `keyring` errors are caught and the supervisor prompts once per session, holding the
secret only in memory.

- _Why keep bindings in `tunnels.toml` not the workspace file?_ Bindings are global to the machine,
  not per window/workspace.

## Risks / Trade-offs

- **`ssh -L` readiness is a heuristic** → poll the local port with a short timeout after spawn and
  treat child exit as failure; `-o ExitOnForwardFailure=yes` makes forward-bind failures fatal
  rather than silent.
- **Host-key prompt blocks a spawned `ssh`** → run with `BatchMode` where possible and surface the
  stderr ("Host key verification failed") as the failure reason so the user can fix `known_hosts`
  out of band; interactive first-connect is a documented manual step for now.
- **`tls_server_name` support / behavior differs across `kube` versions** → D4's confirmation task
  gates the rest; if it does not hold, fall back to a documented `kube` version pin.
- **Paused watches can mask a genuinely dead cluster** → the panel shows a paused state with the
  reason and elapsed time; the user can disconnect to force release.
- **Keychain absent on minimal Linux** → per-session prompt fallback; documented, not fatal.
- **Child `ssh` processes leak on crash** → supervisor holds the `Child`; a process-group kill on
  drop and a startup sweep for orphaned forwards owned by a stale pid file.
- **Scope creep into a port-forward manager** → `K8sPortForward` ships without UI; a task checks no
  user-facing port-forward surface crept in.

## Migration Plan

Additive on top of `bootstrap-fernrohr`. No data migration: absence of `tunnels.toml` means no
tunnels and every context connects directly, exactly as before this change. Rollback is removing the
feature; existing direct connections are unaffected.

## Open Questions

- Whether to supervise `ssh` via a pid file or rely solely on the held `Child` handle - resolved in
  implementation; does not affect specs.
- Exact askpass shim mechanism for feeding a keychain passphrase to `ssh` non-interactively
  (`SSH_ASKPASS` + `SSH_ASKPASS_REQUIRE` vs a helper binary) - implementation detail, spec only
  requires "no prompt when the secret is in the keychain."
