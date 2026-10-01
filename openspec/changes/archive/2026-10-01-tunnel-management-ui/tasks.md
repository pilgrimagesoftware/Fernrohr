# Tasks

## 1. Schema and store

- [x] 1.1 Remove `remote_host`/`remote_port` from `TunnelConfig` and add `auth: TunnelAuth`
  (`SshConfig` default, `KeychainKey`). Verify: a test loads a legacy `tunnels.toml` carrying both
  target fields, asserts the tunnel and bindings load, saves, and asserts neither field is written;
  `no_secret_fields` still passes.
- [x] 1.2 Add `TunnelStore` validation (non-empty host and user, port 1-65535) returning a per-field
  error, plus `usage_counts()` (tunnel id -> bound context count) and `stale_bindings(&[context])`.
  Verify: store tests for each invalid field, for counts across shared bindings, and for a binding
  whose context is absent from the given list - `TunnelStoreError::Invalid(Vec<TunnelFieldError>)`, `usage_counts()`, `stale_bindings(&[String])`.

## 2. Forward target and registry key

- [x] 2.1 Add `kubeconfig::server_for_context(path, context)` returning host and port, defaulting the
  port from the scheme, and an error naming the URL when there is no host. Verify: fixture tests for
  `https://10.0.0.1` (-> 443), `https://api.internal:6443`, and a hostless URL.
- [x] 2.2 Make `ForwardRegistry` generic over its key and add `ForwardKey { tunnel_id, host, port }`
  for SSH tunnels; publish the live key set through a `watch` channel updated on first acquire and
  last release. Verify: registry tests for two acquires of one key sharing an entry, two keys with the
  same `tunnel_id` making two entries, and the watched set on acquire and drop.
- [x] 2.3 Rewire `acquire_for_context` to derive the target via 2.1 and key by `ForwardKey`; a
  hostless server fails the connect with that error and starts no `ssh`. Verify: the existing
  local-sshd connect tests pass with the target taken from a fixture kubeconfig, plus a test that
  two contexts on different servers through one tunnel produce two forwards - a bound context whose target can't be resolved now fails its connect (`ConnectionState::Failed`) instead of silently connecting direct; `acquire_for_context` gained a `kubeconfig_path` argument for fixtures.

## 3. Context-side binding

- [x] 3.1 Add `ui/picker_tunnel.rs`: a per-row tunnel selector (Direct plus each tunnel by name)
  that writes through `TunnelStore::bind`/`unbind` and shows the bound tunnel's name on the row.
  Verify: a GPUI test selects a tunnel for a row and asserts `tunnels.toml` holds the binding and the
  row label updates; selecting Direct removes it - `ui/picker_tunnel.rs`. Manual review found the dropdown didn't pick up a tunnel created in the Tunnels window; fixed with a `TunnelsRevision` global every UI write bumps and every picker and the Tunnels window observe (`ui/tunnels/revision.rs`).
- [x] 3.2 Register a `context.set_tunnel` command opening the same choice for the current context.
  Verify: a command-registry test that the command exists with a title and its handler binds - the dialog is a flat button list, since `Root::open_dialog` rebuilds its content per render and can't own a search widget's state.
- [x] 3.3 Confirm a binding change never alters a live connection. Verify: a test connects through
  tunnel A (fake forward), rebinds to B, asserts the live connection still holds A's forward, then
  reconnects and asserts B is acquired.

## 4. Tunnels window

- [x] 4.1 Add `ui/tunnels/{mod,list}.rs`: a single-instance Tunnels window with the tunnel list
  (name, host, usage count, running state from 2.2's watched set) and a stale-bindings section with a
  per-row Remove. Opened by a `tunnels.manage` command, the app menu, and a "Manage tunnels…"
  control in the picker. Verify: GPUI tests for a second open focusing the existing window, a
  running state that follows a fake forward's acquire and release, and Remove deleting a stale
  binding - also adds the app's first menu ("Fernrohr" > "Manage Tunnels…"); an unreadable kubeconfig shows no stale bindings rather than flagging every binding stale.
- [x] 4.2 Add `ui/tunnels/editor.rs`: create, edit, and rename with 1.2's field errors inline, auth
  choice (with key import into the keychain for `KeychainKey`), jump hosts, and Delete with a confirm
  listing the contexts that fall back to Direct. Verify: GPUI tests for creating a tunnel, renaming
  it with bindings intact, a rejected invalid port, and delete-confirm text naming the bound
  contexts - tunnel ids are generated (`tunnel-<timestamp>`) on first save, never asked for.
- [x] 4.3 Add Test: run the tunnel's `ssh` argv without `-N -L`, with `-o BatchMode=yes -o
  ConnectTimeout=15` and remote command `true`, then report success or the stderr tail. Verify: an
  sshd-integration test for success against the local sshd and failure against a closed port,
  asserting no `ForwardRegistry` entry is created.

## 5. Full verification

- [x] 5.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` all
  pass - 279 passed, 2 ignored. The real-keychain tests are now opt-in (`cargo test -- --ignored`; macOS CI runs them with `--include-ignored`); every other test uses an in-memory secret store under `cfg(test)`.
- [x] 5.2 Manual check against the QA IAP bastions: starting from an empty `tunnels.toml`, create
  `qa-bastion` in the Tunnels window pointing at the `fernrohr-qa-bastion` SSH alias, Test it, then
  set `cluster-a` and `cluster-b` to use it from the picker (no `qa-bastion-ops` needed), connect
  both, and confirm two `ssh` forwards through one bastion, both clusters connected, and the Tunnels
  window showing `qa-bastion` in use by two contexts and running - Paul verified on 2026-09-29: one `qa-bastion` tunnel created from an empty `tunnels.toml`, Test reachable, both contexts bound from the picker, two `ssh` forwards (one per API server) through the one bastion, Tunnels window showing two contexts and Running.