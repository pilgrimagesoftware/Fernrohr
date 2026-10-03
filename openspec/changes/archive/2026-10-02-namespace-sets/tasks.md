# Tasks

## 1. Set storage

- [x] 1.1 Add `config/namespaces.rs` with `NamespaceSetsConfig { sets: Vec<NamespaceSetConfig { name, namespaces }> }` plus the small pure helpers the rest of the change needs: `create` (non-empty name, unique, seeded namespaces), `find`, `apply_to` (add/remove one namespace), `remove_set`, and the sorted, order-insensitive match the title bar uses to name a set; verify each helper with unit tests covering the duplicate-name, empty-set and namespace-absent-from-cluster cases.
- [x] 1.2 Load the file from `preference_dir()/namespace-sets.toml` through the existing `config::load`/`config::save` at app startup, verify first run writes an empty file and that a file that fails to parse leaves the file untouched and yields no sets.

## 2. Commands

- [x] 2.1 Register `namespace_sets.create_set`, `namespace_sets.edit_set`, `namespace_sets.remove_set` and `namespace_sets.switch` in the new `ui/namespace_sets.rs` module with stable ids, so each appears in the command palette, accepts a `keymap.toml` override by id, and shows its resolved key as a hint; verify the ids appear in a generated `keymap.toml` and that a rebind changes the working key after restart.
- [x] 2.2 `create_set` opens the editor prefilled with the focused panel's namespaces and a name field; verify saving produces a set, a duplicate name is refused with a message, a set with no namespaces is refused, and dismissing the editor creates nothing.

## 3. Set editor

- [x] 3.1 Build the editor dialog: a name field, the cluster's filterable namespace list marked as in-set or not, and Save; verify typing filters the list as `namespace-picker-filter` specifies and that Escape clears the filter before closing.
- [x] 3.2 List namespaces the saved set holds that the connected cluster does not have, marked absent, and allow removing them; verify a set can be edited in a cluster where one of its namespaces does not exist.
- [x] 3.3 Save add and remove immediately through `config::save` and reload; verify a set edited across two launches holds the edits.
- [x] 3.4 `remove_set` lists the saved sets, asks for confirmation naming the set, and on confirm deletes it and saves; verify cancelling leaves the set in place and confirming removes it from the file.

## 4. Quick selection

- [x] 4.1 Open the quick-selection picker over the saved sets in file order, showing digits `1`-`9` beside the first nine only while a namespaced panel has focus; verify a tenth set is listed with no digit.
- [x] 4.2 Apply the set when its digit is pressed, and on Enter for the highlighted row; verify a digit switches the focused panel and that a set past the ninth is reachable by arrows and Enter.
- [x] 4.3 Close on Escape without changing any scope, and report plainly when opened without a namespaced panel in focus; verify both.

## 5. Applying a set

- [x] 5.1 Applying a set to the focused panel sets its scope to `IncludeExclude { include, names }` and persists with the panel; verify a panel switched to a two-namespace set shows both and only those, and that a namespace absent from the cluster stays in the panel's scope and in the set.
- [x] 5.2 Extract `k8s::cluster::apply_scope_to_context(context_name, scope)` from `warp-all-to-namespace`'s propagation and route its "warp all" command through it, so both changes share one definition of "for all".
- [x] 5.3 Add the context-wide apply command on top of that helper: every open namespaced panel in the active context takes the set and the context default becomes it, so a panel opened later in that context starts on the set; verify with a two-panel test and verify panels in another context and cluster-scoped panels are unchanged.

## 6. Title bar and wiring

- [x] 6.1 Have the namespace scope indicator show a matched set's name instead of the count, following the exact-match rule; verify a hand-built scope and a set edited away from under a panel both read as a count.
- [x] 6.2 Register the module from `util/shell/app.rs`, so every command is available in a window, and check the affected title-bar and dialog files stay under the ~500-line cap by splitting them if not.

## 7. Verification

- [x] 7.1 Cover the whole feature from the keyboard: create a set, edit it, switch to it by digit, and apply it to the context, using a `TestAppContext` keyboard test and mocked namespace list, with no live-cluster dependency.
- [x] 7.2 Run `cargo test`, `cargo clippy` and `cargo fmt --check` in the App workspace and verify the pre-existing namespace scoping tests still pass unchanged.
