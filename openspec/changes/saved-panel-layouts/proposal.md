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
  current window's arrangement under a name, and load a saved layout back by name in one of two
  ways - Add, which opens the saved layout's panels alongside whatever the window already shows,
  or Replace, which swaps the window's current arrangement for the saved one. There is no
  "restore into a new window" mode: both ways of loading a saved layout act on the current window.
- **What a saved layout captures**: the dock tree (splits, panel sizes, tab order, active tab,
  zoom), the Resource panel's width and visibility, each panel's content key (kind, object,
  namespace selection, and cluster context), each list panel's filter text and sort column and
  direction, and the window's size and position.
- **One file per saved layout**, under a new `layouts/` folder in the application's state
  directory, rather than one file holding every saved layout. A layout's filename is derived from
  its display name; the display name itself is stored inside the file, so sanitizing or
  disambiguating the filename never changes what the user sees. A single unreadable layout file is
  skipped and reported, without affecting any other saved layout.
- **Restoring never silently drops state.** A panel whose saved layout points at a cluster context
  the window does not currently hold restores as a placeholder naming the problem, rather than
  being dropped, silently connecting a context behind the user's back, or erroring the whole load.
  A namespace or object that is gone, or a panel kind this build does not recognize, is handled the
  same way the automatic restore already handles each case today.
- **A Settings "Layouts" section** lists every saved layout and lets the user remove one. Removing
  a saved layout is irreversible (it cannot be recovered), so it asks through the application's
  existing irreversible-confirmation dialog - opening with focus on Cancel, so Enter cancels, and
  requiring a deliberate action to actually delete. The section is keyboard-operable the same way
  the existing Settings sections (Keyboard Shortcuts, Appearance, Panels) are.
- **Fully keyboard-operable**, per the project's keyboard-first rule: Save, Load (Add), and Load
  (Replace) are registered commands with default keybindings and Window-menu entries where one
  fits; the saved-layouts picker is operable end-to-end from the keyboard, with hover never moving
  its selection; renaming a saved layout stays reachable from the picker.
- **Secret values are never saved.** A revealed `SecretValue` has no `Serialize` impl today
  (`object-detail`'s existing guarantee); saved layouts rely on that same structural guarantee
  rather than adding a parallel redaction step.

## Capabilities

### New Capabilities
- `saved-panel-layouts`: named, user-managed panel-arrangement snapshots - save, list, load by
  name (adding to, or replacing, the current window's arrangement), rename, and delete - each
  stored as its own file, layered on top of the existing automatic per-window restore.

### Modified Capabilities

None. The existing `app-shell` requirement to persist and restore a window's last state on
relaunch is unchanged; this change adds a second, user-named store next to it, and reuses
`command-system`'s existing registry/palette/keymap/menu mechanism, the application's existing
irreversible-confirmation dialog, and `object-detail`'s existing "a `SecretValue` cannot
serialize" guarantee, without changing any of their requirements.

## Impact

- App code (for design.md to detail): a new per-layout file store under
  `state_dir()/layouts/` (`app/src/util/paths.rs`), next to `app/src/util/shell/persist.rs`'s
  `workspace.toml` and `app/src/config/dock_layouts.rs`'s `dock-layouts.json`; new commands
  registered alongside `app/src/util/shell/app.rs`'s existing ones; a new picker view alongside
  `app/src/ui/picker/`; a new section in `app/src/ui/settings.rs`; reuse of
  `app/src/ui/confirm_dialog.rs`'s existing irreversible-confirmation dialog.
- A new atomic-write pattern (temp file + rename) for this one store: no existing persistence
  code in the app writes atomically today, so this introduces that pattern rather than reusing one.
- No new external dependencies: reuses the existing `serde_json`, `dirs`, and `gpui_kit` dock-state
  machinery already used for the automatic restore.
