## 1. Integration verification

- [ ] 1.1 End to end on macOS and Linux: define two tunnels, bind three contexts (two sharing one
      tunnel, one on the other, one unbound), connect all four, confirm two underlying `ssh` forwards
      and one direct connection, and confirm `tunnels.toml` holds no secrets - **macOS passed
      2026-09-29; Linux still open.** Against the Kochava QA IAP bastions: `tunnels.toml` defined
      `qa-bastion` (`ko-q-network-cin-uw1a-bastion-c64h` -> greedygoat `10.193.223.242:443`) and
      `qa-bastion-ops` (`...-bastion-ops-8fj4` -> carefulcrab `10.193.15.242:443`); `bastion_host`
      is a `~/.ssh/config` alias whose `ProxyCommand` is `gcloud compute start-iap-tunnel ... 22
      --listen-on-stdin`, so the app's own `ssh -N -L` rides IAP with no app change. `greedygoat`
      and its `gke_..._greedygoat` alias both bind `qa-bastion`; `carefulcrab` binds
      `qa-bastion-ops`; `ire-1` (public endpoint) is unbound. All four connected; `ps` showed
      exactly two `ssh -N -L` children (one per tunnel id, the two greedygoat contexts sharing one
      via `ForwardRegistry`), `lsof` showed tunneled contexts talking only to their loopback
      forwards and `ire-1` directly to `35.227.164.133:443`, and `tunnels.toml` contains no key
      matching the `no_secret_fields` pattern
- [x] 1.2 Flap test: kill one bastion's `ssh` forward mid-session, confirm the two dependent
      connections pause and their panels stay open, restore the bastion, confirm both resume and
      their Pods tables reconverge - killed `qa-bastion`'s process group and held it down 12s by
      killing each respawn; the supervisor's retries landed 1s/2s/4s apart (backoff as designed),
      the first ~4s after the kill (the 10s health-check tick), and the forward came back ~8s
      after release on the **same** local port, so the rewritten `cluster_url` stayed valid. Both
      greedygoat panels stayed open showing "paused, reconnecting", then cleared and their Pods tables
      refilled after the reconnect with no panel reopened; `carefulcrab` and `ire-1` were untouched
- [ ] 1.3 Credential test against a cluster using an exec plugin (or a faithful stand-in): force a
      token expiry, confirm re-auth resumes watches with no panel loss
- [x] 1.4 Confirm TLS: connect a bastioned context and assert the connection validates against the
      real API server host with no insecure flag anywhere in the path - GKE private endpoints are
      IPs, so `rewrite_for_tunnel` pins `tls_server_name` to a bare IP; the certs carry it as an IP
      SAN. Through the app's own forwards, `curl --cacert <kubeconfig CA> --connect-to
      <ip>:443:127.0.0.1:<port>` verified both clusters (`ssl_verify_result=0`, HTTP 200) and a
      wrong-IP control failed with a SAN mismatch. No `insecure-skip-tls-verify` in the three
      clusters' kubeconfig entries, and no insecure verifier anywhere in `app/src`

(Moved verbatim from `tunnel-subsystem`'s section 8 — deferred because this machine has no bastion
host, no second OS, and no exec-plugin-backed cluster to test against.)

## 2. Findings (follow-up changes, not this one)

- Tunnel and binding UI: `tunnel-subsystem` 5.3/6.1 shipped only the `TunnelStore` layer, so
  tunnels and bindings are hand-edited TOML today. The model also ties a tunnel to one
  `remote_host`, so one bastion serving many clusters needs one tunnel entry per cluster; a
  follow-up should split bastion (SSH details) from forward target (derived from the context's
  kubeconfig `server:`), and bindings keyed on context name silently drop on a context rename.
- Paused/reconnecting state belongs in a status bar, colored to draw attention, not inline in each
  panel.
- IAP connect takes ~4s, close to the 5s `SSH_READINESS_PROBE_TIMEOUT`; a slow network could
  mark a healthy-but-slow forward failed and burn an extra backoff cycle.
