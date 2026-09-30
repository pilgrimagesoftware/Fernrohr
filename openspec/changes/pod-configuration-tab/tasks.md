# Tasks

Depends on `resource-links` (archive it first, so `object-detail` is in `openspec/specs` for the
MODIFIED requirement above to apply to).

## 1. Secret values

- [ ] 1.1 Add `SecretValue` (decoded bytes; `Debug`/`Display` print `<secret: N bytes>`; one
  `expose()`; no `Serialize`). Verify: tests assert `format!("{:?}")` and `format!("{}")` print
  no fixture value.
- [ ] 1.2 A reveal fetch: fresh `get` of one Secret, keep one key's decoded value as a
  `SecretValue`, drop the rest. Verify: a fixture-server test reveals one key, and asserts the other
  key's value isn't held anywhere in the result.

## 2. Configuration tab

- [ ] 2.1 Project the tab's entries from the pod's existing references: one per object, first-seen
  order, with every use. Verify: tests for a Secret used by a volume and by `valueFrom` (one entry,
  two uses), projected sources, and image pull secrets.
- [ ] 2.2 Add `DetailSection::Configuration` with positional tab keys (1-6) and fetch the entries'
  objects when the tab is first shown, per card: loaded, not found, or failed. Verify: a render test
  that nothing is fetched before the tab is shown, and `simulate_keystrokes` tests for `3` and the
  shifted keys.
- [ ] 2.3 Render the cards (link, uses, ConfigMap key/values, Secret keys and sizes) and the reveal
  buttons (tab stops; Enter/Space toggle; hidden on tab switch and panel close), plus the Hide
  Secret Values command. Verify: `simulate_keystrokes` tests for Tab + Space revealing one value,
  a tab switch hiding it, and `h` hiding all; a test that the dock layout dump holds no value.

## 3. Object viewer

- [ ] 3.1 Give the Secret section the same per-key reveal and command, keeping the YAML redacted.
  Verify: reveal tests as in 2.3, and a test that the YAML has no value while one is revealed.

## 4. Verification

- [ ] 4.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, `cargo test`.
- [ ] 4.2 Manual check against a cluster: open a pod's Configuration tab, reveal and hide a Secret
  value by mouse and by keyboard, confirm the YAML stays redacted.
  **Needs user confirmation against a running build.**
