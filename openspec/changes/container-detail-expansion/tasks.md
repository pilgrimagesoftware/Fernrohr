# Tasks

## 1. Extended container data

- [ ] 1.1 Extend `summarize_containers`'s output with the fields expansion needs: env (literal
  values verbatim, `value_from` as a reference string), volume mounts, probes (one summarized
  line each), command/args, security context - computed unconditionally alongside the existing
  summary fields (design.md: the data is already in hand, no second fetch). Verify: a test with a
  container carrying a literal env var, a Secret-sourced one, a volume mount, and a readiness
  probe asserts each renders correctly, and the Secret-sourced value never appears.

## 2. Expand/collapse UI

- [ ] 2.1 Each container card gets an expand/collapse control, using the same label-keyed
  `open_sections` state the Managed Fields/Tolerations/Volumes disclosures already use (keyed by
  container name, not field label, to stay distinct from those). Verify: a test expands one
  container and asserts only that container's extended fields render.

## 3. Full verification

- [ ] 3.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` all
  pass.
- [ ] 3.2 Manual smoke test: expand a container with env vars, volume mounts, and a probe;
  confirm a Secret-sourced env var shows its reference, not its value.
