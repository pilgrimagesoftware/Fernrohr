# Tasks

## 1. Config schema

- [ ] 1.1 Add `TunnelKind` (`ssh` default | `command`), `CommandTunnelMode` (`proxy` default |
      `forward`) and `CommandTunnelConfig` (`command_line`, `mode`, `local_port: Option<u16>`,
      `startup_timeout_secs` default 30) to `config/tunnels.rs`, with SSH fields unchanged at the
      top level. Verify that unit tests round-trip a command tunnel through TOML, load a pre-change
      file as `ssh` with bindings intact, and keep `no_secret_fields` passing.
- [ ] 1.2 Add `shlex` as a direct dependency and a `tunnel::command::argv` module that strips
      backslash-newline, splits with POSIX quoting, and substitutes `{port}` per argument after
      splitting. Verify with unit tests for quoted arguments, a pasted multi-line command,
      unbalanced quotes (error), multiple `{port}`s, and a port value that can't alter splitting.
- [ ] 1.3 Add command-tunnel validation to the tunnel store's save path. It must reject an empty
      command, unbalanced quotes, no `{port}` without a fixed port, a port outside 1-65535, and a
      non-positive timeout, each with a field-specific reason. Verify with store unit tests per
      rule.

## 2. Command transport

- [ ] 2.1 Add `util::login_env` to resolve and cache the login-shell `PATH`, with a 5 s timeout,
      delimited capture and fallback directories. Verify with unit tests for the parser and the
      fallback, using an injected runner.
- [ ] 2.2 Implement `tunnel::command::CommandTransport: ForwardTransport`. It spawns argv in its own
      process group with the login `PATH`, holds stdin open, keeps a 50-line output ring buffer,
      polls TCP readiness, and on startup timeout kills the group and returns a failure with the
      output tail. Verify with tests that use a small test-helper listener binary, or `nc -l`
      guarded by availability, to cover: ready, exit-before-ready, and timeout.
- [ ] 2.3 Implement stop: TERM to the process group, a 3 s grace period, then KILL. Implement
      health: child alive and a TCP connect. Verify with a test that spawns a command whose child
      spawns a grandchild listener, and asserts both are gone and the port is free after release.
      Add a test that a killed child drives the supervisor to Reconnecting and back to Up on the
      same port.
- [ ] 2.4 Extend `util::pidfile` with a `cmd-pids` directory recording the argv[0] basename. The
      startup and quit sweeps verify that basename instead of the `ssh -N -L` signature. Verify
      with pidfile unit tests that a matching orphan is swept and a recycled unrelated pid is not.

## 3. Registry and connection routing

- [ ] 3.1 Turn `ForwardKey` into `Ssh { tunnel_id, host, port }` | `Command { tunnel_id }`, and let
      the registry hold either transport. Command tunnels skip the server-URL target derivation.
      Verify with a registry test that two contexts on different API servers bound to one command
      tunnel share one forward, and that the existing SSH sharing tests still pass.
- [ ] 3.2 Enable `kube`'s `http-proxy` feature. In `connection/connect.rs`, a proxy-mode tunnel
      sets `config.proxy_url = http://127.0.0.1:<port>` and leaves `cluster_url` and
      `tls_server_name` untouched, and forward mode keeps the rewrite. Verify with a unit test of
      the config-building function for each of SSH, forward and proxy, asserting the URL, TLS
      server name and proxy URL.
- [ ] 3.3 Add an integration test that runs a minimal in-process HTTP `CONNECT` proxy as the
      "tunnel" and a stub HTTPS API server. Build a client through proxy mode, and assert that a
      request reaches the stub through the proxy with TLS validated against the stub's own name.
      Assert that an unbound client in the same test does not touch the proxy.

## 4. Tunnels panel UI

- [ ] 4.1 Add a kind switch to the tunnel editor that shows the SSH form or the command form
      (multi-line command, Proxy/Forward mode with help text, optional local port, startup timeout,
      and a plain-text-storage hint), with inline validation errors from 1.3. Keep both forms'
      values when switching kinds. Verify with editor view tests for switching kinds, saving each
      kind, and showing each validation error. The editor must be fully keyboard-operable.
- [ ] 4.2 Show a kind badge in the tunnel list. Route Test for command tunnels through a one-shot
      `CommandTransport` start, then stop, reporting success or the output tail. Verify with a view
      test for the badge, and a test-flow test that leaves no process running after either
      outcome.
- [ ] 4.3 Document command tunnels in the user docs (or the README tunnels section), with a
      placeholder-only example such as `gcloud compute ssh <host> --tunnel-through-iap --project
      <project> -- -N -L{port}:127.0.0.1:8888`, and update `docs/architecture.md` and
      `openspec/config.yaml`'s tunnel paragraph together. Verify that the two stay in sync and
      contain no real infrastructure identifiers.

## 5. Gates

- [ ] 5.1 `cargo fmt -- --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` all
      pass, and new or touched files stay under the 500-line limit.

## 6. Verification

- [ ] 6.1 Manual check: create a proxy-mode command tunnel from a real IAP/bastion command with
      `{port}`, bind a context, and connect. Confirm that resources list and logs stream.
      **Needs user confirmation.**
- [ ] 6.2 Manual check: quit Fernrohr with the tunnel Up, and confirm with `ps` that no `gcloud` or
      `ssh` from it remains. Relaunch and reconnect. **Needs user confirmation.**
- [ ] 6.3 Manual check: open a pod shell and a port-forward through the proxy-mode tunnel, which
      covers websocket upgrades over `CONNECT`. **Needs user confirmation.**
