# Design

## Context

Tunnels are `TunnelConfig` entries in `tunnels.toml`, with a `TunnelKind` of `Ssh` or `Command`.
Each kind's settings are flattened into the config so that switching kind keeps the other kind's
fields. Connecting a bound context calls `acquire_for_context`, which acquires a refcounted
forward from `ForwardRegistry` under a `ForwardKey`. That forward's `ForwardSupervisor` runs a
`ForwardTransport` on Tokio. `connect_and_probe` puts the connection in
`ConnectionState::WaitingForTunnel` and waits, with no timeout, for the forward to report `Up`.
When the last connection drops its `RegistryHandle`, the forward is torn down.

The registry lock is held only for lookup and insert, so tunnels never wait on each other. The
status bar and cluster picker already render `WaitingForTunnel`. Three things are missing: there
is no way to cancel a wait, every existing kind spawns and supervises a process, and the app has
no desktop notification support.

See [proposal.md](proposal.md), `manual-tunnel`, and `desktop-notifications` for behavior.

## Goals / Non-Goals

**Goals:**

- Add a manual kind that fits the existing acquire, share, and release lifecycle, so every other
  part of connection handling stays unchanged.
- Hold only the contexts bound to the waiting tunnel. The UI and every other context keep running.
- Give the user a prompt they notice even when Fernrohr is in the background, and that works fully
  from the keyboard.

**Non-Goals:**

- Detecting or controlling the VPN client itself, or watching for the VPN dropping after it has
  been confirmed. The app has no reliable view into an external network path. A dropped VPN shows
  up as ordinary connection interruptions.
- A general cancel for waiting SSH or command tunnels.
- Notifications for anything other than manual tunnels in this change.
- Windows-specific notification work. The kind itself works on every platform.

## Decisions

### D1. Manual kind as a flattened settings table

Add `TunnelKind::Manual` and a flattened `manual` table holding `message: Option<String>` and
`skip_when_reachable: bool`, which defaults to `true`. This follows the command-tunnel approach, so
switching kind keeps every kind's settings and files without the table still load. The editor shows
a message field and a skip-when-reachable toggle, and the tunnel list shows a manual badge.

### D2. A transport that waits on the user, keyed by tunnel

Add `ForwardKey::Manual { tunnel_id }`. As with command tunnels, the key is the tunnel alone, so
every context bound to the tunnel shares one forward and therefore one confirmation. The new
`ManualTransport::connect` publishes a pending confirmation and then waits on a channel that
Proceed or Cancel resolves. Proceed returns `Ok`, which makes the forward `Up`. `health_check`
always succeeds, because there is nothing for the app to supervise. The forward rewrites no
address: `acquire_for_context` returns the handle so the connection waits on it, but the kube
`Config` keeps the context's own server.

Reusing the registry and supervisor gives sharing, the waiting state, and release on the last
disconnect without new code paths. A separate gate outside the forward registry was considered.
It would duplicate refcounting and the waiting-state plumbing, and need its own release rules.

### D3. Cancel is a terminal transport failure

The supervisor retries failed connects with backoff, which is wrong for a user's Cancel. Add a
non-retryable error outcome to `ForwardTransport::connect`. When it is returned, the supervisor
closes the forward's state channel with that reason instead of retrying. `connect_and_probe`
already fails the connection when the channel closes before `Up`; it now uses the transport's
reason ("`corp-vpn` was cancelled"). That fails every connection sharing the forward. They then
drop their handles, which removes the forward, so the next attempt prompts again. SSH and command
transports never return the new outcome, so their behavior is unchanged.

### D4. Pending confirmations model

A global GPUI entity, `ManualConfirmations`, lists the pending prompts. Each entry holds the tunnel
id, the display name, the message, the waiting context names, and a resolver for its channel.
`ManualTransport` adds an entry by crossing to the foreground. The status bar, the picker, and the
Proceed and Cancel commands observe and resolve entries. The connection state stays
`WaitingForTunnel`. `ContextHealth` gains an `AwaitingConfirmation` variant, chosen when the
context's tunnel has a pending entry, so the status bar can show distinct text and controls
without a new `ConnectionState`. Resolving an entry removes it. Dropping the forward (all
connections closed while waiting) also removes it, so a prompt never outlives its waiters.

### D5. Reachability shortcut

When `skip_when_reachable` is on, `ManualTransport::connect` first tries a TCP connect to the
acquiring context's API server host and port, with a 2-second timeout, on Tokio. If it succeeds, it
returns `Ok` without publishing a prompt. Because the key is the tunnel, the first acquiring
context's server decides for every context sharing it. That suits the use case, where one VPN
covers every cluster behind it. A successful TCP connect does not prove the right network path; a
public endpoint would pass it. That is why the setting can be turned off.

### D6. Desktop notifications through `notify-rust`

A small `notify` module exposes `post(title, body, on_activate)`. It is backed by `notify-rust`,
which uses the macOS notification center and the freedesktop D-Bus service on Linux, and can be
swapped out behind the module. Posting runs on a blocking Tokio task, so the UI thread never waits
on it. Failures are logged at `warn` and otherwise ignored, because the status bar and picker
carry the prompt regardless. Activation focuses the window that started the waiting connection,
where the platform reports clicks. On macOS, a build without a bundle identifier, such as a
`cargo run` development build, may not show notifications, and the spec accepts that because the
in-app prompt remains.

One notification is posted per pending entry, when the entry is created. Contexts that join an
existing entry do not post again.

### D7. Commands and controls

Register two commands, `tunnel.manual.proceed` ("Proceed with Manual Tunnel") and
`tunnel.manual.cancel` ("Cancel Manual Tunnel"), in the global key context, each with a
keymap-overridable default binding. With one pending entry they act on it directly. With several,
they open a picker of pending tunnels that follows the picker selection model. The status-bar item
gets Proceed and Cancel icon buttons with tooltips that show the bound keys, and the cluster
picker's status line gets the same controls. Cancel is recoverable, since connecting again prompts
again, so it needs no confirmation dialog.

## Risks / Trade-offs

- [The VPN drops after it was confirmed] -> Kubernetes requests fail through the existing
  interruption handling. The confirmation stays until the last context disconnects, so reconnecting
  after an interruption does not prompt again. Accepted, because the app cannot observe the VPN.
- [The skip-when-reachable probe passes without the intended network path] -> The setting can be
  turned off per tunnel.
- [Notifications are suppressed by OS settings or Focus modes] -> The status-bar and picker prompts
  stay visible until resolved.
- [A user forgets a waiting prompt] -> The wait has no timeout, by request. The status-bar item
  shows how long it has been waiting, and the connection can be cancelled or the window closed.
- [`notify-rust` on macOS depends on a deprecated notification API] -> It is isolated behind the
  `notify` module so it can be swapped for `UNUserNotificationCenter` once builds are signed and
  bundled.
