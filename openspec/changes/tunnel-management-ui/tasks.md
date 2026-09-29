# Tasks

## 1. Schema and store

- [ ] 1.1 Remove `remote_host`/`remote_port` from `TunnelConfig` and add `auth: TunnelAuth`
  (`SshConfig` default, `KeychainKey`). Verify: a test loads a legacy `tunnels.toml` carrying both
  target fields, asserts the tunnel and bindings load, saves, and asserts neither field is written;
  `no_secret_fields` still passes.
- [ ] 1.2 Add `TunnelStore` validation (non-empty host and user, port 1-65535) returning a per-field
  error, plus `usage_counts()` (tunnel id -> bound context count) and `stale_bindings(&[context])`.
  Verify: store tests for each invalid field, for counts across shared bindings, and for a binding
  whose context is absent from the given list.

## 2. Forward target and registry key

- [ ] 2.1 Add `kubeconfig::server_for_context(path, context)` returning host and port, defaulting the
  port from the scheme, and an error naming the URL when there is no host. Verify: fixture tests for
  `https://10.0.0.1` (-> 443), `https://api.internal:6443`, and a hostless URL.
- [ ] 2.2 Make `ForwardRegistry` generic over its key and add `ForwardKey { tunnel_id, host, port }`
  for SSH tunnels; publish the live key set through a `watch` channel updated on first acquire and
  last release. Verify: registry tests for two acquires of one key sharing an entry, two keys with the
  same `tunnel_id` making two entries, and the watched set on acquire and drop.
- [ ] 2.3 Rewire `acquire_for_context` to derive the target via 2.1 and key by `ForwardKey`; a
  hostless server fails the connect with that error and starts no `ssh`. Verify: the existing
  local-sshd connect tests pass with the target taken from a fixture kubeconfig, plus a test that
  two contexts on different servers through one tunnel produce two forwards.

## 3. Context-side binding

- [ ] 3.1 Add `ui/picker_tunnel.rs`: a per-row tunnel selector (Direct plus each tunnel by name)
  that writes through `TunnelStore::bind`/`unbind` and shows the bound tunnel's name on the row.
  Verify: a GPUI test selects a tunnel for a row and asserts `tunnels.toml` holds the binding and the
  row label updates; selecting Direct removes it.
- [ ] 3.2 Register a `context.set_tunnel` command opening the same choice for the current context.
  Verify: a command-registry test that the command exists with a title and its handler binds.
- [ ] 3.3 Confirm a binding change never alters a live connection. Verify: a test connects through
  tunnel A (fake forward), rebinds to B, asserts the live connection still holds A's forward, then
  reconnects and asserts B is acquired.

## 4. Tunnels window

- [ ] 4.1 Add `ui/tunnels/{mod,list}.rs`: a single-instance Tunnels window with the tunnel list
  (name, host, usage count, running state from 2.2's watched set) and a stale-bindings section with a
  per-row Remove. Opened by a `tunnels.manage` command, the app menu, and a "Manage tunnels…"
  control in the picker. Verify: GPUI tests for a second open focusing the existing window, a
  running state that follows a fake forward's acquire and release, and Remove deleting a stale
  binding.
- [ ] 4.2 Add `ui/tunnels/editor.rs`: create, edit, and rename with 1.2's field errors inline, auth
  choice (with key import into the keychain for `KeychainKey`), jump hosts, and Delete with a confirm
  listing the contexts that fall back to Direct. Verify: GPUI tests for creating a tunnel, renaming
  it with bindings intact, a rejected invalid port, and delete-confirm text naming the bound
  contexts.
- [ ] 4.3 Add Test: run the tunnel's `ssh` argv without `-N -L`, with `-o BatchMode=yes -o
  ConnectTimeout=15` and remote command `true`, then report success or the stderr tail. Verify: an
  sshd-integration test for success against the local sshd and failure against a closed port,
  asserting no `ForwardRegistry` entry is created.

## 5. Full verification

- [ ] 5.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` all
  pass.
- [ ] 5.2 Manual check against the QA IAP bastions: starting from an empty `tunnels.toml`, create
  `qa-bastion` in the Tunnels window pointing at the `fernrohr-qa-bastion` SSH alias, Test it, then
  set `greedygoat` and `carefulcrab` to use it from the picker (no `qa-bastion-ops` needed), connect
  both, and confirm two `ssh` forwards through one bastion, both clusters connected, and the Tunnels
  window showing `qa-bastion` in use by two contexts and running.
