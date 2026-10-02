# Proposal

## Why

Namespace scoping is a per-panel value that has to be retyped on every panel and every window.
`namespace-include-exclude` makes that value a set instead of a single name, so a panel can watch
"team-a, team-b, team-c" - but it is still anonymous: the set lives in the panel, is not named, and
is not reusable. The set a user builds for staging triage is thrown away when the panel closes, and
re-entering it in the next window is pure typing. Meanwhile `warp-all-to-namespace` already
propagates a scope across every panel in a context, so a set that could be *named* would propagate
just as well - the only missing piece is a name and somewhere to keep it.

## What Changes

- **Named namespace sets**: a user-defined, named list of namespaces that a panel can be switched
  to in one action. Creating a set seeds it from the focused panel's current namespace scope, so
  the common case is "name what I'm already looking at".
- **A set editor** to add and remove namespaces from a set, and to create new sets and delete
  existing ones. The namespace list it offers is the connected cluster's own list, filterable as in
  `namespace-picker-filter`, and it is fully keyboard-operable.
- **Quick selection**: one command opens a picker listing every set with a digit beside each; typing
  that digit switches to it, so switching is two keystrokes from anywhere a panel has focus.
- Switching applies to the focused panel, or - via a companion command - to every namespaced panel
  in the window's active context and to that context's default, reusing
  `warp-all-to-namespace`'s existing propagation helper.
- Sets persist across launches in a user-editable preferences file, so they follow the user between
  clusters and windows rather than living in one workspace file.

Not in scope: making a set the unit of storage for a panel's scope (a panel keeps the resolved
namespace names, not a reference - see design.md decision 4), per-cluster sets, and sharing a set
between clusters by name with different contents.

## Capabilities

### New Capabilities

- `namespace-sets`: user-defined named sets of namespaces, the editor that creates and edits them,
  and switching a panel or a context to one.

### Modified Capabilities

- `resource-browser`: a panel's namespace scope reports the saved set it matches, so the title bar
  names the set instead of a count when one applies.

## Impact

- App code:
  - `App/app/src/config/namespaces.rs`: a new `namespace-sets.toml` preference file -
    `config::NamespaceSetsConfig { sets: Vec<NamespaceSetConfig> }`, ordered so a set's position is
    its quick-selection digit, loaded and saved through the existing `config::load`/`save` pair in
    `preference_dir()` (same treatment as `keymap.toml`, which is also a user's own file).
  - `App/app/src/ui/namespace_sets.rs` (new, plus `picker.rs` and `editor.rs` beside it): the four
    commands and the two dialogs. New module because `ui/menu.rs` and `util/shell` are both past the
    file-size cap.
  - `App/app/src/ui/panel/title.rs`: the namespace-scope indicator (`namespace-include-exclude`
    task 4.1's "N included") reports a set's name when the panel's scope matches one exactly.
  - `App/app/src/util/shell/app.rs`: registers the new commands, so each is a palette entry, gets a
    `keymap.toml` override by id, and a menu item.
- Depends on `namespace-include-exclude` for the `IncludeExclude` scope this change switches a panel
  to, on `warp-all-to-namespace` for cross-panel propagation, and on `namespace-picker-filter` for
  the filterable namespace list the editor reuses. `namespace-include-exclude` and
  `warp-all-to-namespace` must land first; `namespace-picker-filter` is a UI reuse only.
- No new dependencies.