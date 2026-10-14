# Tasks

## 1. Typed panel identity

- [ ] 1.1 Give `LabelLogs::Typed` a `TypedId` (D1): a random non-zero `u64` from `getrandom` per
  new panel, carried in the target and so in its `PanelKey`. Update every match on
  `LabelLogs::Typed`. Verify with a unit test in `util/logs/labels/tests.rs`: two new ids differ
  and neither is 0.
- [ ] 1.2 Deserialize `label_logs` from both forms (D2): `{"Typed": <id>}`, and the old bare
  `"Typed"` as `TypedId(0)`, with workload sources unchanged. Verify with unit tests: each form
  round-trips or reads as expected, and `labels_from_state` and `panels::panel_key` read an old
  state to the same target.

## 2. Opening and switching

- [ ] 2.1 Make Open Logs by Label… always open a new empty typed panel in the focused list's
  context and namespaces (D3). Verify with a shell test in `util/shell/label_logs_tests/` using
  real keystrokes: open, type `app=web`, open again; there are two typed panels, the second empty
  with its selector field focused, and the first still streaming. Update the existing tests that
  expected a reopen to focus.
- [ ] 2.2 Check that two typed panels in one context stream independently and their titles name
  their selectors. Verify with a shell test: `app=web` and `app=worker` panels each stream only
  their own pods, and the titles read `Logs: app=web` and `Logs: app=worker`.
- [ ] 2.3 Remove the focus-the-other-panel branch from `switch_label_logs_context` (D4). The
  replacement keeps the switched panel's `TypedId`. Verify by updating
  `label_logs_tests/context.rs`'s "context with its own typed panel" test: after the switch, both
  panels search `other`, the switched one in its tab following `app=web`, the other still
  following its selector, and focus on the switched panel.

## 3. Saved layouts

- [ ] 3.1 Check that a layout with two typed panels in one context restores both, each with its
  own selector. Verify with a shell test that dumps the dock with two typed panels and loads it
  back.
- [ ] 3.2 Check that a layout saved before this change, with `"label_logs": "Typed"`, restores its
  typed panel with its selector. Verify with a restore test over a fixture state in the old form.

## 4. Documentation

- [ ] 4.1 Update `docs/site/latest/logs-shells.md`'s "Follow pods by label": opening it again opens
  another panel, several can be open in one context, and switching context moves the panel even
  when another already searches that context. Use placeholder names only. Verify: the docs-site
  check (`bundle exec ruby _scripts/check.rb`) and `cargo test` pass.

## 5. Integration

- [ ] 5.1 Run `cargo fmt -- --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test`
  in `App/`. Verify: all pass.
- [ ] 5.2 Check manually: open Logs by Label twice in one context with different selectors, and
  once in another context. Switch one panel to a context another already searches. Save the layout,
  reload it, and check that every panel comes back with its own selector and streams. Record no
  cluster-identifying details.
