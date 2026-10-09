# Proposal

## Why

Some clusters are only reachable through a VPN that a person has to bring up by hand, from a
menu-bar client that has no CLI or automation hook. Fernrohr's SSH and command tunnels can only
gate a connection on something the app starts itself. Today, connecting to such a cluster before
the VPN is up just fails or hangs until the network times out, and nothing reminds the user to
switch the VPN on first.

## What Changes

- Add a third tunnel kind, **manual**, that starts nothing. When a context bound to it connects,
  Fernrohr asks the user to bring up the external network path and holds that context's connection
  until the user chooses Proceed or Cancel.
- A manual tunnel carries an optional instruction message, for example "Connect the corporate VPN
  in the menu bar", shown in every prompt. It also has a reachability shortcut, on by default:
  when the context's API server already answers a TCP connect, the prompt is skipped.
- The prompt appears in three places: a desktop notification, a status-bar item with Proceed and
  Cancel controls, and the cluster picker's status line. Proceed and Cancel are also
  command-palette commands.
- The wait never blocks the UI or any other context's connection. Contexts bound to the same
  manual tunnel share one prompt, so one Proceed releases all of them. Cancel fails each waiting
  connection with a clear reason, and never falls back to a direct connection.
- A confirmation lasts while any connected context is using the tunnel. When the last one
  disconnects, the tunnel is released and the next connection prompts again.
- Add desktop notification support to the app. Manual tunnels are its first user; nothing else
  sends notifications in this change.

## Capabilities

### New Capabilities

- `manual-tunnel`: the manual tunnel kind's settings, its prompt and confirmation lifecycle,
  Proceed and Cancel, the reachability shortcut, and how waiting contexts are shown.
- `desktop-notifications`: posting OS desktop notifications from the app, and the in-app behavior
  when the platform refuses or cannot show them.

### Modified Capabilities

- `tunnel-config`: tunnels gain the manual kind alongside SSH and command tunnels in the Tunnels
  panel and editor.
- `context-tunnel-binding`: a manual tunnel derives no forward target and rewrites nothing. All
  contexts bound to one manual tunnel share a single confirmation and connect directly once it is
  confirmed.

## Impact

- Tunnel configuration model, `tunnels.toml` persistence, and the tunnel editor and list.
- Tunnel acquisition and the forward registry. A manual kind needs a transport that waits on a
  user signal, and connections need a way to cancel a wait that is in progress.
- Connection state and context health in the status bar and the cluster picker.
- A new desktop-notification dependency, with platform-specific behavior on macOS and Linux.
- Command registry: new Proceed and Cancel commands.
