## 1. Integration verification

- [ ] 1.1 End to end on macOS and Linux: define two tunnels, bind three contexts (two sharing one
      tunnel, one on the other, one unbound), connect all four, confirm two underlying `ssh` forwards
      and one direct connection, and confirm `tunnels.toml` holds no secrets
- [ ] 1.2 Flap test: kill one bastion's `ssh` forward mid-session, confirm the two dependent
      connections pause and their panels stay open, restore the bastion, confirm both resume and
      their Pods tables reconverge
- [ ] 1.3 Credential test against a cluster using an exec plugin (or a faithful stand-in): force a
      token expiry, confirm re-auth resumes watches with no panel loss
- [ ] 1.4 Confirm TLS: connect a bastioned context and assert the connection validates against the
      real API server host with no insecure flag anywhere in the path

(Moved verbatim from `tunnel-subsystem`'s section 8 — deferred because this machine has no bastion
host, no second OS, and no exec-plugin-backed cluster to test against.)
