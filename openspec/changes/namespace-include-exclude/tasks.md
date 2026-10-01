# Tasks

## 1. NamespaceScope data model

- [ ] 1.1 Add `IncludeExcludeMode { Include, Exclude }` and `NamespaceScope::IncludeExclude { mode, namespaces: BTreeSet<String> }` in `App/app/src/config/workspace.rs`, and verify `cargo test -p app config::workspace` passes including round-trip-through-TOML for the new variant
- [ ] 1.2 Add `NamespaceScope` helper methods: `include(&mut self, ns)`, `exclude(&mut self, ns)` (collapsing an empty exclude set back to `All`, per design.md), and `matches(&self, ns: &str) -> bool`, and verify unit tests cover: include into existing include set, exclude into `All`, removing the last excluded namespace collapses to `All`, exclude composed against `Single`/include falls back to "all minus excluded" per design.md
- [ ] 1.3 Update `matches_namespaces` (`App/app/src/k8s/resource/pods/rows.rs` per earlier grep) to handle the new variant, and verify existing namespace-filter tests plus new include/exclude cases pass

## 2. Commands

- [ ] 2.1 Add `IncludeNamespace` / `ExcludeNamespace` actions and commands (`pods/commands.rs`), scoped to `PANEL_KEY_CONTEXT`, with default keybindings and `MenuSlot::View` entries, and verify they appear in the command palette and View menu while a Pods panel has focus
- [ ] 2.2 Wire the handlers in `pods.rs` to call `NamespaceScope::include`/`exclude` with the selected row's namespace on the panel's current scope, and verify a keystroke test (`VisualTestContext::simulate_keystrokes`) drives each command end to end
- [ ] 2.3 Add `IncludeNamespaceForAll` / `ExcludeNamespaceForAll` commands that call `warp-all-to-namespace`'s per-context propagation helper with the include/exclude mutation instead of a scope replacement, and verify a test with two Pods panels in the same context confirms both update and a new panel opened afterward inherits the set

## 3. Persistence and restore

- [ ] 3.1 Confirm `PanelDescriptor::Pods.namespace` round-trips the new variant through `workspace.toml` save/restore, and verify with a restore test that opens a panel with an `IncludeExclude` scope and reloads it
- [ ] 3.2 Verify an older `workspace.toml` fixture with only `All`/`Single` values still loads without error (backward compatibility), via a fixture-based test

## 4. UI feedback

- [ ] 4.1 Update the Pods panel's namespace indicator/hint row to show "N included" / "N excluded" (or similar) when scope is `IncludeExclude`, and verify a snapshot/unit test asserts the label text for each mode
- [ ] 4.2 Verify `cargo clippy -p app -- -D warnings` and `cargo fmt -- --check` pass after all changes
