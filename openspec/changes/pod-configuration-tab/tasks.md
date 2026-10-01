# Tasks

Depends on `resource-links` (archived 2026-09-30, meta #35), so `object-detail` is in
`openspec/specs` for the RENAMED/MODIFIED delta to apply to.

As built (App branch `1-secret-values`), one addition beyond the tasks: switching tab by keyboard
keeps focus in the panel when the focused control (a reveal button) unmounts with its tab, so the
next key isn't lost. The reveal state (`secret_value::Reveal`) and the key row
(`ui::detail::secret_key_row`) are shared by both panels.

## 1. Secret values

- [x] 1.1 Add `SecretValue` (decoded bytes; `Debug`/`Display` print `<secret: N bytes>`; one
  `expose()`; no `Serialize`). Verify: tests assert `format!("{:?}")` and `format!("{}")` print
  no fixture value.
- [x] 1.2 A reveal fetch: fresh `get` of one Secret, keep one key's decoded value as a
  `SecretValue`, drop the rest. Verify: a fixture-server test reveals one key, and asserts the other
  key's value isn't held anywhere in the result.

## 2. Configuration tab

- [x] 2.1 Project the tab's entries from the pod's existing references: one per object, first-seen
  order, with every use. Verify: tests for a Secret used by a volume and by `valueFrom` (one entry,
  two uses), projected sources, and image pull secrets.
- [x] 2.2 Add `DetailSection::Configuration` with positional tab keys (1-6) and fetch the entries'
  objects when the tab is first shown, per card: loaded, not found, or failed. Verify: a render test
  that nothing is fetched before the tab is shown, and `simulate_keystrokes` tests for `3` and the
  shifted keys.
- [x] 2.3 Render the cards (link, uses, ConfigMap key/values, Secret keys and sizes) and the reveal
  buttons (tab stops; Enter/Space toggle; hidden on tab switch and panel close), plus the Hide
  Secret Values command. Verify: `simulate_keystrokes` tests for Tab + Space revealing one value,
  a tab switch hiding it, and `h` hiding all; a test that the dock layout dump holds no value.

## 3. Object viewer

- [x] 3.1 Give the Secret section the same per-key reveal and command, keeping the YAML redacted.
  Verify: reveal tests as in 2.3, and a test that the YAML has no value while one is revealed.

## 4. Verification

- [x] 4.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, `cargo test`.
- [x] 4.2 Manual check against a cluster: open a pod's Configuration tab, reveal and hide a Secret
  value by mouse and by keyboard, confirm the YAML stays redacted - confirmed by the user
  2026-10-01.

## 5. Collapsed large values (amendment)

Added 2026-10-01 at the user's request after 4.2: long values made the cards hard to scan.

- [x] 5.1 Treat a ConfigMap value as large when it is longer than 20 characters or spans more than
  one line, and render it collapsed by default as its first 20 characters of the first line plus
  an ellipsis, with an expand/collapse control (tab stop; Enter/Space toggle; click). Secret values
  are never collapsed: they stay hidden until revealed and show in full when revealed. Verify:
  render tests that a long and a multi-line ConfigMap value show only the preview, that a short
  value has no control, and `simulate_keystrokes` tests that Tab + Space expands one value and
  leaves the others collapsed.
- [x] 5.2 Reset ConfigMap values to collapsed when the tab is shown again or the panel closes, and
  keep Secret keys free of any collapse control. Verify: a tab-switch test, and tests that a Secret
  key has no collapse control whether hidden or revealed and that a revealed long Secret value is
  shown in full.
- [ ] 5.4 Place the expand control directly beside the key's name rather than at the row's far
  edge (from the 5.3 check: it was easy to miss on a wide panel). Verify: a layout test that the
  control sits beside the key in a wide panel.
- [ ] 5.3 Manual check: open a pod with a long ConfigMap value and a long Secret value; confirm the
  ConfigMap value starts collapsed showing a short preview and expands and collapses by mouse and
  keyboard, and the Secret value starts hidden and shows in full when revealed.
