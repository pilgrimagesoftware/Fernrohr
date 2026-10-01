# Proposal

## Why

Fernrohr already restores a window's panels on the next launch, but that memory is singular and
implicit: one window holds one remembered state, overwritten the moment the user rearranges it.
Someone who builds a "triage" layout (Pods + Logs side by side on a staging context) and a
"deploy" layout (a different context's Pods plus Events) has no way to keep both - opening one
erases the other. Named, user-saved arrangements let a layout a user built once be called back
deliberately, as many times as needed, independent of whatever the window currently shows.

## What Changes

- **Named saved layouts**, separate from the automatic last-session restore: the user can save the
  current window's arrangement under a name, list saved layouts, restore one (replacing the
  current window's arrangement, or into a new window), rename, overwrite, and delete.
- **What a saved layout captures**: the dock tree (splits, panel sizes, tab order, active tab,
  zoom), the Resource panel's width and visibility, each panel's content key (kind, object,
  namespace, and cluster context), and the window's size and position.
- **Restoring never silently drops state.** A saved context that isn't currently connected is
  connected on demand; a context that no longer exists in the kubeconfig, or a namespace/object a
  panel pointed at that no longer exists, becomes a placeholder panel with a reconnect or
  not-found affordance instead of being dropped or erroring.
- **Fully keyboard-operable**, per the project's keyboard-first rule: Save and Manage/Restore are
  registered commands with default keybindings and Window-menu entries; the management/restore
  picker is operable end-to-end from the keyboard, with hover never moving its selection.
- **Secret values are never saved.** A revealed `SecretValue` has no `Serialize` impl today
  (`object-detail`'s existing guarantee); saved layouts rely on that same structural guarantee
  rather than adding a parallel redaction step.

## Capabilities

### New Capabilities
- `saved-panel-layouts`: named, user-managed panel-arrangement snapshots - save, list, restore
  (in place or into a new window), rename, overwrite, and delete - layered on top of the existing
  automatic per-window restore.

### Modified Capabilities

None. The existing `app-shell` requirement to persist and restore a window's last state on
relaunch is unchanged; this change adds a second, user-named store next to it, and reuses
`command-system`'s existing registry/palette/keymap/menu mechanism and `object-detail`'s existing
"a `SecretValue` cannot serialize" guarantee without changing either's requirements.

## Impact

- App code (for design.md to detail): a new saved-layouts store next to
  `app/src/util/shell/persist.rs`'s `workspace.toml` and `app/src/config/dock_layouts.rs`'s
  `dock-layouts.json`; new commands registered alongside `app/src/util/shell/app.rs`'s existing
  ones; a new picker view alongside `app/src/ui/picker/`.
- No new external dependencies: reuses the existing `toml`/`serde_json`, `dirs`, and
  `gpui_kit` dock-state machinery already used for the automatic restore.
