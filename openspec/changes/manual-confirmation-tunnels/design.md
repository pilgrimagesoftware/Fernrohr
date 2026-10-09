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
an Instruction field (blank saves as no message) and a reachability choice, "Skip the prompt" or
"Always prompt", as a two-button switch like the command form's mode. A manual tunnel has no Test
action, since there is nothing to start. The tunnel list shows a Manual badge and, as the summary,
the instruction (truncated) or "Confirmed by hand".

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

On Linux, a click is reported through the D-Bus default action and focuses the originating window. On macOS, the notification posts under the app's bundle identifier (otherwise it appears to come from Finder) and has no click callback, since waiting for one would block the main run loop. `notify-rust` is a Unix-only dependency, and Windows logs that notifications are unsupported.

One notification is posted per pending entry, when the entry is created. Contexts that join an
existing entry do not post again.

### D7. Commands and controls

Register two commands, `tunnel.manual.proceed` ("Proceed with Manual Tunnel") and
`tunnel.manual.cancel` ("Cancel Manual Tunnel"), in the global key context, each with a
keymap-overridable default binding. With one pending entry they act on it directly. With several,
they open a picker of pending tunnels that follows the picker selection model. The cluster
picker's status line gets Proceed and Cancel controls. Cancel is recoverable, since connecting
again prompts again, so it needs no confirmation dialog.

### D8. The waiting context's own capsule carries the prompt

There is no separate status-bar item. The waiting context already has a status-bar capsule, drawn
by `render_capsule` from its `ContextHealth`, so `AwaitingConfirmation` changes how that capsule
looks and what its menu offers:

- **Look:** the same rounded, bordered capsule layout as every other context (see
  `status-capsule-icons`: name, `[tunnel]`, state icon, elapsed time). A new `Severity::Attention` fills the capsule with the theme's warning color and draws its
  contents in the matching foreground. Every other state only tints the text. A filled capsule
  stays the most visible thing in the row, and still reads as a context. The state icon is an
  attention icon from the existing set, distinct from every other state's icon. Its tooltip reads
  "Awaiting confirmation", with the elapsed wait and the tunnel's message, so a long message never
  widens the status bar.
- **Menu:** the chevron's dropdown lists Proceed and Cancel, a separator, then the existing
  Disconnect. Proceed and Cancel call the same `ManualConfirmations` resolution as the commands,
  and their menu rows show the commands' bound keys. Clicking the capsule body still activates the
  context, as it does for every other capsule.
- **Sorting:** capsules already sort non-connected first. Attention sorts ahead of every other
  state, so a waiting context is never pushed off the visible end of the row.

Every context waiting on the same tunnel gets this treatment, and resolving from any of them
resolves the shared entry (D4). Disconnect on a waiting capsule only closes that context. It does
not cancel the tunnel for others; if it was the last waiter, the entry goes away with the forward.

Separate Proceed and Cancel icon buttons were considered. They would add a second control style
next to every other capsule's single menu, and widen the row while waiting.

### D9. Implementation notes from the lifecycle work

- The non-retryable outcome is `ForwardTransport::connect_outcome() -> Result<(), ConnectFailure>`
  with `Retry` and `GiveUp`. Its default wraps `connect()` as `Retry`, so the SSH, command, and
  pod port-forward transports are unchanged. On `GiveUp`, the supervisor records the reason and
  ends, which closes the state channel; `connect_and_probe` fails with that reason.
- Manual tunnels use a new `TunnelRoute::Direct`: no address rewrite and no proxy.
- A connection whose route is `Direct` releases its forward handle when it fails. That is what
  makes Cancel, and a failure after Proceed, withdraw the confirmation so the next attempt prompts
  again. SSH and command forwards keep their handles on failure, as before.
- `ManualConfirmations` (in `tunnel::manual`) exposes `pending()`, `pending_for(tunnel_id)`,
  `for_context(name)`, and `resolve(tunnel_id, Proceed | Cancel)`. An entry's `contexts` is
  recorded at acquire time; a context that disconnects while others keep waiting stays listed
  until the entry resolves, so the UI reads each context's own connection state too.

### D10. Implementation notes from the prompt UI

- `ContextHealth::AwaitingConfirmation` applies only while the context's own connection is
  `WaitingForTunnel` and the pending entry lists it; Failed and Paused take precedence. Its icon is
  `BellRing`, used by no other state.
- The commands are `tunnel.manual.proceed` (⌘⌥P) and `tunnel.manual.cancel` (⌘⌥C), in the
  Context menu after Manage Tunnels. A capsule's menu rows show those keys but answer that
  capsule's own tunnel directly, never opening the picker.
- `ManualConfirmations` starts at launch and emits a `Prompted` event once per new entry, which
  the notification posting subscribes to. A notification click focuses the active main window,
  or the first one: the app does not record which window started a connection, so "the window
  that started the waiting connection" is approximated.

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
