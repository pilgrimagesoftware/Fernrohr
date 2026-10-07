# Design

## Context

The automatic restore already in place (`app/src/util/shell/persist.rs`,
`app/src/config/workspace.rs`, `app/src/config/dock_layouts.rs`) is the foundation this builds on:

- `state_dir()` (`app/src/util/paths.rs`) is `dirs::data_dir()/<app_dir>/` - window state and
  workspace layout live there today (`workspace.toml`, `dock-layouts.json`); a sibling
  `preference_dir()` (`dirs::preference_dir()/<app_dir>/`) holds user-editable files
  (`keymap.toml`, `ui.toml`). Saved layouts are machine-written state the user triggers
  explicitly, so they belong under `state_dir()`, not `preference_dir()`.
- `WorkspaceConfig { windows: Vec<WindowLayout> }` persists to `state_dir()/workspace.toml`
  (TOML, via the generic `config::load`/`config::save`). `WindowLayout` holds window bounds
  (`width`, `height`, `x`, `y`), the window's `contexts: Vec<String>`, and a
  `resource_panel_width: Option<f32>` (`None` is hidden; `Some(width)` is visible at that width -
  there is no separate visibility field). `layout_from_window`/`layout_from_bounds`
  (`util/shell/layout.rs`) build one from a live window; `window_bounds` turns one back into
  `Bounds<Pixels>`, falling back to `Bounds::centered` when a saved position no longer applies.
- `state_dir()/dock-layouts.json` (JSON, via `serde_json`) stores `DockLayouts { layouts:
  HashMap<String, DockAreaState> }`, keyed by `context_lifecycle::dock_layout_key(contexts)`.
  `DockAreaState` is `gpui_kit`'s own dock-dump type: it already carries the split tree, pane
  sizes, tab order, active tab, zoom, and each panel's opaque `data` field. `MainWindow::
  save_dock_layout` (`util/shell/window.rs`) produces one from a live window with
  `dock_area.read(cx).dump(cx)`; `DockArea::load` is the matching rebuild path a window already
  uses at open time.
- Both the TOML and the JSON store follow the same convention: a parse failure returns
  `T::default()` without touching the file (`config::load`'s `unwrap_or_default()` /
  `dock_layouts::load`'s `.ok()...unwrap_or_default()`), and `save` does a plain
  `fs::create_dir_all` + `fs::write` - **no store in the app writes atomically today**; a search
  for `tempfile`/`rename(`/`.tmp` across `app/src` finds nothing. Atomic writes are new
  infrastructure this change introduces, not a pattern it reuses.
- A saved panel's content key - its kind, object, namespace, and cluster context - already has a
  decoder: `PanelKey { target: NavTarget, context_name: String, namespaces: Vec<String> }` and
  `restored_panel_keys(state: &PanelState) -> Vec<Option<PanelKey>>` live in
  `app/src/util/shell/panels.rs` (not `ui/nav.rs`, which only defines `NavTarget` itself), and are
  `pub(super)` - visible only within `util::shell`. `panel_key()` matches on
  `state.panel_name` ("Pods", "Logs", "Exec", "PodDetail", "ObjectDetail", "ObjectList"/"Resource",
  "Events") and reads fields like `data["context_name"]` out of the panel's raw `DockAreaState`
  JSON; an unrecognized `panel_name` or a missing required field yields `None`.
- A panel's namespace selection is already in its key (`PanelKey.namespaces`), so saving it needs
  nothing new. A list panel's *view state* - its filter text and its sort column and direction -
  is not persisted anywhere today (the automatic restore drops it too). This change adds it to
  each list panel's `DockAreaState` panel `data` as optional fields, where the panel has that
  view state at all:
  - **ObjectList**: `filter` text and `sort: { column, descending }`.
  - **Pods**: `sort: { column, descending }` only - the Pods panel has no filter input
    (`pods/rows.rs`'s `matches_filter`/`view_rows` are unwired), so there is no filter to save.
  - **Events**: `filter` (its search text). Its sort was already persisted before this change, in
    its own tri-state shape `sort: { column, order: "ascending" | "descending" | "default" }`
    (`events_browser/restore.rs`); that shape is kept, not reshaped, so files already written
    still decode.

  The fields are written on save and applied after the panel is built on load (an ObjectList
  builds its filter input before it reads its rows, so a restored filter applies on the first
  frame). They stay optional, so older saved files and the automatic restore's
  `dock-layouts.json` still decode, and an unknown sort column is ignored rather than failing the
  panel. The automatic restore picks up the same fields as a side effect.
- A panel whose saved state can't be restored already has a placeholder, not a dropped tab or a
  crash: `ui::unrestored::restore_with` wraps each panel kind's own restore function and, on
  `Err(reason)`, logs it and shows an `UnrestoredPanel` - a panel that names its own kind and the
  reason, keeps its original state unchanged (so a later build or a corrected file can still
  restore it), and does nothing else (no retry or reconnect action exists on it; the user closes
  it like any panel). This is the same mechanism the automatic restore already falls back to for
  an unknown panel kind.
- `MainWindow::open_target_in` (`util/shell/open.rs`) is the one path every panel open goes
  through, already deduplicating by content key (opening an already-open target focuses it
  instead of adding a second copy) - and it **refuses** a `context_name` the window does not
  currently hold rather than connecting or substituting one. There is no "connect a context on
  demand while opening a panel" path to reuse; a window's held contexts are only ever grown
  through its own explicit add-context control.
- Commands are `Command { id, title, default_binding, context: Option<&str>, action, menu:
  Option<MenuSlot> }` in a `CommandRegistry` global (`app/src/command.rs`); `keymap.rs` derives
  `keymap.toml` defaults from the registry, and `keymap::conflicts` checks for colliding bindings
  *within the same active `KeyContext` stack* - a binding scoped to one context never collides
  with the same binding scoped to a different, unrelated context.
- `app/src/ui/confirm_dialog.rs` is the application's one confirmation mechanism, already built
  for exactly the Irreversible/Recoverable split this change needs: `confirm_dialog::open`,
  given a `Confirmation { title, body, confirm, id_prefix, severity }`, draws Cancel and a
  danger-styled confirm button, each showing its live key. At `Severity::Irreversible`, it moves
  focus to Cancel once the dialog draws (so bare Enter cancels) and the confirm button fires only
  on click, Tab+Enter/Space, or the already-registered `dialog.confirm_irreversible` command
  (default `secondary-backspace`, i.e. `cmd-backspace` on macOS / `ctrl-backspace` elsewhere) - no
  new binding is needed for the confirm step itself. Full tunnel deletion
  (`ui/tunnels/editor/actions.rs::request_delete`) is the existing precedent for "ask through this
  dialog, at `Severity::Irreversible`, before a destructive remove."
- `ui/settings.rs`'s `SettingsWindow` holds a `Section` enum (`KeyboardShortcuts` default,
  `Appearance`, `Panels`), a sidebar of tab-stop buttons that call `show(section, ...)` on
  click or Enter/Space, and a per-section `FocusHandle` that `show` focuses so the next Tab lands
  inside the shown section. Each section also has a `ShowX` command (no default binding - a
  `cmd-<digit>` would collide with the global show-panel keys), scoped to context
  `"SettingsWindow"`.
- `ui/picker/` (`ClusterPicker`: `state.rs`, `interaction.rs`, `layout.rs`, `rows.rs`,
  `render.rs`) is the existing keyboard-first, searchable-list reference: `render.rs` tells a
  keyboard-driven highlight from mouse hover with `window.last_input_was_keyboard()`, only calling
  `follow_keyboard` for a hover that keyboard input didn't cause.
- `SecretValue` (`app/src/k8s/resource/secret_value.rs`) has no `Serialize` impl by construction;
  nothing this change does can put a revealed value into a saved layout, because there is no path
  for it to reach serde at all. `object-detail`'s own spec already has a matching scenario ("A
  revealed value stays out of saved state") for the window's layout being saved.

See proposal.md for motivation.

## Goals / Non-Goals

**Goals:**
- Add a second, named, user-managed layout store, one file per layout, that sits next to the
  automatic single-slot store without changing its behavior.
- Reuse the existing dock-dump (`DockAreaState`) and panel-key machinery rather than inventing a
  parallel representation of "what a panel shows".
- Make every new user-facing action a registered command: palette entry, default binding where one
  makes sense, Window-menu item where one fits.
- Never lose a panel on restore, even when its context, namespace, or object is gone, or its kind
  is one this build doesn't recognize.
- Give Settings a "Layouts" section, keyboard-operable like its existing sections, that lists and
  can remove saved layouts, with removal going through the app's existing irreversible-confirmation
  dialog.

**Non-Goals:**
- Restoring a saved layout into a new window. Dropped per explicit requirement: Add and Replace,
  both acting on the current window, are what's required, and no "open a new window, cascaded from
  the last one" mechanism exists for `shell.new_window` (`cmd-n`) to reuse - adding that mode here
  would be new window-placement work the proposal doesn't ask for, not something that falls out
  for free.
- Connecting a cluster context "on demand" while loading a saved layout. `open_target_in` already
  refuses a context the window doesn't hold rather than connecting one; this change keeps that
  refusal and shows a placeholder instead, rather than adding a new auto-connect path.
- Per-workspace keymap overrides (`architecture.md` already defers this; out of scope here).
- Syncing saved layouts across machines or users.
- Saving/restoring anything from the Resource panel's content beyond its width and visibility
  (its own scroll position, filter text, etc. are out of scope - the same granularity the
  automatic restore already stops at).
- Changing `object-detail`'s Secret-redaction requirement, `confirm_dialog`'s severity mechanism,
  or `ui::unrestored`'s placeholder mechanism; this change only depends on each of them.
- A retry or "pick a replacement context" affordance on a restored placeholder. No such affordance
  exists on `UnrestoredPanel` or on `pod-detail`'s existing "pod no longer exists" state today;
  this change gives a saved-layout placeholder the same shape those already have (name the
  problem, stay open until the user closes it) rather than adding new interactive recovery that
  nothing else in the app offers yet.

## Decisions

### D1: A new capability, not a modified `app-shell`
`app-shell`'s requirement ("persist, per window... and restore them on the next launch") describes
the *automatic, implicit, singular* restore and stays true unchanged. Named saves are a distinct,
user-initiated, multi-slot concept layered on top, so they get their own capability
(`saved-panel-layouts`) rather than reworded `app-shell` text.

### D2: One file per layout, under `state_dir()/layouts/`
Each saved layout is its own file, `state_dir()/layouts/<derived-filename>.json`, rather than one
file holding a list. A layout's own content (`DockAreaState`, which already round-trips through
`serde_json` in `dock-layouts.json`) can be large; one file per layout means a corrupt or
truncated save can only ever cost that one layout, never the whole collection, and a save never
has to rewrite every other layout's bytes to add or update one.

```rust
/// One saved layout's on-disk shape; the whole content of one file under
/// `layouts/`.
struct SavedLayout {
    version: u32,            // schema version; bump on breaking shape changes
    name: String,            // the display name the user gave it; source of truth,
                              // independent of the file's own (derived, sanitized) name
    created_at: String,      // RFC 3339
    updated_at: String,      // bumped on overwrite and rename
    contexts: Vec<String>,   // same shape as WindowLayout.contexts
    dock: DockAreaState,     // same type dock-layouts.json already stores
    resource_panel_width: Option<f32>, // same meaning as WindowLayout's field: None is
                                         // hidden, Some(width) is visible at that width
    window_width: f32,
    window_height: f32,
}
```

**Filename derivation.** A display name is slugified for its filename: lowercase, runs of
characters outside `[a-z0-9]` collapsed to a single `-`, leading/trailing `-` trimmed; a name that
slugifies to nothing (all punctuation, or non-ASCII with no ASCII fallback) uses a fixed
placeholder stem. Two distinct display names can still slugify to the same stem (`"My Layout"` and
`"my_layout"` both give `my-layout`); on save, if the derived filename already exists on disk under
a *different* saved name, a numeric suffix (`-2`, `-3`, ...) is appended until the filename is
free. The display name always lives inside the file (`SavedLayout.name`), so this sanitizing and
disambiguation never surfaces to the user - the picker and Settings section display `name`, never
the filename.

**Name uniqueness is case-insensitive** (`"Deploy"` and `"deploy"` collide as the same saved
name), which keeps the common case - names differing only by case or by punctuation a slug drops -
on the "saving over an existing name asks to overwrite" path (already specified) instead of
silently becoming two files with suffixed, diverging filenames.

**Renaming** re-derives the filename from the new name the same way. When the derived filename
changes, the rename writes the new file first (atomically, see below) and removes the old one only
after the new file is on disk, so a crash mid-rename leaves the old file as the recoverable copy
rather than losing both.

**Atomic write.** Saving a layout writes to a temp file in the same `layouts/` directory, then
`fs::rename`s it onto the final filename - a same-filesystem rename is atomic on both macOS and
Linux. This is new infrastructure (see Context: no existing store does this); it is scoped to this
one store rather than retrofitted onto `workspace.toml` or `dock-layouts.json`, which are out of
scope for this change.

**Reading the collection.** Listing saved layouts reads every `*.json` file in `layouts/`
independently: a file that fails to read or parse is skipped and reported by its filename (not a
guessed display name, since a file that doesn't parse has none to offer) as unreadable, without
touching that file or affecting any other layout's listing. The `layouts/` directory itself is
created (`fs::create_dir_all`) the first time a layout is saved, not eagerly at startup - mirroring
`dock_layouts.rs::save`'s existing `create_dir_all(parent)` call.

**Alternative considered:** one `saved-layouts.json` holding every layout (the prior version of
this design). Rejected: it reintroduces the single-file failure mode this change exists to avoid -
a corrupt collection file loses every saved layout at once, and every save rewrites every other
layout's bytes - and it also complicates the explicit "corrupt file doesn't affect the others"
requirement, which falls out naturally once each layout is its own file.

### D3: Content-key reuse, and where the new code lives
A saved layout's `dock: DockAreaState` carries panel `data` in the same shape
`restored_panel_keys()`/`panel_key()` already decode into `PanelKey`/`NavTarget`. Because that
decoder is `pub(super)` inside `util::shell` (by design - see `rust-structure.md`'s "widen only as
far as the move requires"), the code that captures a live window into a `SavedLayout` and applies
one back (Add/Replace) is a new sibling module, `app/src/util/shell/saved_layouts.rs`, so it calls
`restored_panel_keys`/`PanelKey` directly with no visibility change needed. The on-disk format and
file I/O (the `SavedLayout` struct itself, filename derivation, atomic save/load/rename/remove) is
a separate, `util::shell`-independent module, `app/src/config/saved_layouts.rs`, mirroring
`dock_layouts.rs`'s placement - it only needs `DockAreaState` and plain data, the same dependency
`dock_layouts.rs` already has.

### D4: Loading a saved layout: Add vs. Replace
Two registered commands scoped to the picker's `KeyContext` (`SavedLayoutsPicker`), mirroring the
existing pattern of panel-scoped commands (`object-detail`'s per-tab, per-action bindings):

- **Replace** (`saved_layouts.load_replace`, default `enter`): closes the window's current panels
  and rebuilds its dock from the saved `DockAreaState` via `DockArea::load` - the same rebuild path
  a window already uses when it opens - then applies the saved `resource_panel_width` and window
  bounds (`window_bounds`'s existing fallback already handles a saved position that no longer fits
  any connected display). This is a wholesale swap: "I asked for *that* layout," not a
  content-only merge.
- **Add** (`saved_layouts.load_add`, default `secondary-enter`, the app's existing cross-platform
  idiom for a modified-Enter variant - e.g. the Pods and Resource-list panels' own
  open-in-background binding): decodes the saved layout's panels into `(NavTarget, context_name,
  namespaces)` via the same decoder D3 reuses, then opens each one through
  `MainWindow::open_target_in` exactly as any other open request would. Because that path already
  deduplicates by content key, adding a layout identical to what's already open changes nothing;
  adding a different one genuinely lays its panels alongside the existing ones. Add does not touch
  the window's Resource panel state or bounds - it is additive only, never a wholesale swap.

Neither mode is a stacked second dialog after the picker confirms: a user holding a key through a
picker to confirm shouldn't hit a modal fork afterward asking which mode, so Add and Replace are
two distinct keys from the start, the same way the picker's `enter` does one specific thing today.

Renaming (`saved_layouts.rename_selected`, default `r`) and deleting
(`saved_layouts.delete_selected`, default `backspace`) stay picker-scoped commands too (see D6 for
delete). None of `cmd-shift-s`, `cmd-shift-o`, `enter`, `secondary-enter`, `r`, or `backspace`
(each of the last four scoped to `SavedLayoutsPicker`) collides with a binding already registered
elsewhere: `r` and `backspace` are each bound bare in other panels' own `KeyContext`s
(`ObjectListPanel`/`EventsPanel`, and `KeyboardShortcuts`, respectively), `secondary-enter` is
bound in `PodsPanel`/`ObjectListPanel`, and none of those contexts is active at the same time as
`SavedLayoutsPicker` - checked by reading each file's `default_binding` at proposal time (`app/src/
command.rs`, `ui/settings/shortcuts.rs`, `k8s/resource/object_list/commands.rs`,
`k8s/resource/events_browser/commands.rs`, `k8s/resource/pods/commands.rs`); `keymap::conflicts`
is still the authoritative, context-aware check at implementation time.

| id | title | default binding | context |
|---|---|---|---|
| `layouts.save` | Save Panel Layout… | `cmd-shift-s` | `Workspace` (not available from the empty cluster picker) |
| `layouts.manage` | Saved Layouts… | `cmd-shift-o` | none (available even from the cluster picker, so a fresh window can load a layout straight away) |
| `saved_layouts.load_replace` | Load (Replace) | `enter` | `SavedLayoutsPicker` |
| `saved_layouts.load_add` | Load (Add) | `secondary-enter` | `SavedLayoutsPicker` |
| `saved_layouts.rename_selected` | Rename | `r` | `SavedLayoutsPicker` |
| `saved_layouts.delete_selected` | Delete | `backspace` | `SavedLayoutsPicker` |
| `settings.show_layouts` | Settings: Show Layouts | `` (no default; a `cmd-<digit>` would collide with global show-panel keys, same as the other `ShowX` section commands) | `SettingsWindow` |

### D5: Restoring never silently drops state
Each saved panel's context is classified against the window's own held contexts
(`WindowMode::Workspace`'s `contexts`, the same list `open_target_in` already checks):

- **Held by this window already** -> build the panel immediately, through the normal open path (D4).
- **Not held by this window** -> whether the context exists in the kubeconfig, is connected
  elsewhere, or doesn't exist at all, the saved panel is **not** opened and **not** silently
  dropped: it restores as a placeholder, in the same dock slot, naming the context it needed and
  that this window isn't connected to it - the same shape `ui::unrestored::UnrestoredPanel` already
  gives an unparseable saved panel (names the problem, offers no retry, closes like any panel).
  This keeps `open_target_in`'s existing refusal intact rather than adding a new "connect a context
  behind the user's back" path (see Non-Goals).

A namespace or object that is gone is a cluster-level `404` the panel discovers once it actually
queries, after it is already open - the same case `pod-detail`'s existing "pod no longer exists"
requirement already covers for pod detail panels ("the panel SHALL show that the pod no longer
exists rather than erroring or closing unexpectedly"). This change relies on that same per-panel-
kind handling rather than adding a new one; it does not add a retry affordance beyond what that
existing requirement already provides.

An unrecognized panel kind in a saved layout's dock JSON - from a newer build that wrote the file -
goes through `ui::unrestored` exactly as the automatic restore's own unknown-panel case already
does: that one slot shows "not restored" and every other panel in the layout restores normally.

### D6: Deleting a saved layout is Irreversible
Deleting a saved layout cannot be recovered (there is no undo, no trash), so it is asked through
the app's existing `confirm_dialog::open` at `Severity::Irreversible` - the same mechanism and the
same tier `ui/tunnels/editor/actions.rs::request_delete` already uses for a full tunnel delete.
Concretely: the dialog opens with focus on Cancel, so a bare Enter cancels; the danger-styled
Delete button fires only on click, on Tab+Enter/Space while it holds focus, or on the already-
registered `dialog.confirm_irreversible` shortcut (`secondary-backspace`) - no new binding is
introduced for the confirm step. Both entry points - the picker's `saved_layouts.delete_selected`
(`backspace`, opens the dialog) and the Settings "Layouts" section's per-row Remove control (an
icon button with a tooltip, per `icon-buttons.md`, also a tab stop) - open the same dialog with the
same severity, so deleting reads, looks, and keys alike from either surface.

### D7: Settings "Layouts" section
A new `Section::Layouts` variant joins `ui/settings.rs`'s existing `KeyboardShortcuts`,
`Appearance`, and `Panels` sections: its own sidebar button, its own `FocusHandle` that `show`
focuses (so Tab lands on the list), and its own module, `app/src/ui/settings/layouts.rs`, following
`panels.rs`'s placement. It lists every saved layout by name (reading the same
`config::saved_layouts` store D3 introduces) with a per-row Remove control going through D6's
confirmation. Per the explicit requirement, this section only lists and removes; renaming stays in
the picker (D4) rather than being duplicated here, so there is exactly one place that edits a saved
layout's name. A `settings.show_layouts` command (D4's table) makes the section reachable from the
palette while the Settings window has focus, matching `settings.show_panels`'s existing pattern
(no default binding, scoped to `"SettingsWindow"`).

### D8: Secret safety is inherited, not re-implemented
Because `SavedLayout.dock: DockAreaState` is produced by the same dock-dump path
`dock-layouts.json` already uses, and `SecretValue` has no `Serialize` impl, a revealed value
cannot reach a saved layout file through any serialization path - there is nothing extra to
redact. A test (mirroring `object-detail`'s own "revealed value stays out of saved state" test)
asserts a `format!("{:?}")` of a saved layout containing a revealed value has no fixture value in
it, as a structural regression guard rather than a behavioral one.

## Risks / Trade-offs

- [A saved layout's dock JSON references panel kinds the current build doesn't know about, from a
  newer version that wrote the file] -> handled by the existing `ui::unrestored` placeholder (D5);
  every other panel still restores.
- [Window size/position in a saved layout no longer fits any connected display (laptop vs. external
  monitor)] -> Replace reuses `layout.rs`'s existing `window_bounds()` clamping/centering fallback,
  which already handles a missing or stale position for the automatic restore.
- [A user deletes a saved layout by mistake] -> `Severity::Irreversible` confirmation (D6); no undo
  is added in this change, matching the lack of undo elsewhere for an Irreversible action today.
- [Two different display names slugify to the same filename] -> numeric-suffix disambiguation on
  save (D2); the display name inside the file is always what the user sees, so this never surfaces.
- [Saving very often (once per task switch) accumulates many small files in `layouts/`] -> no
  retention limit in this change; left as a follow-up if it proves to matter in practice.
- [A rename's filename change is interrupted mid-write] -> the new file is written (atomically)
  before the old one is removed (D2), so a crash leaves the old file - the saved layout survives
  under its previous filename, recoverable on next load even though the rename didn't complete.

## Migration Plan

Purely additive: a new directory, new file format, new commands, a new menu entry, a new Settings
section. No existing file format, command id, or keybinding changes. First run with no `layouts/`
directory behaves exactly as today until the user saves a layout for the first time (the directory
is created then, not eagerly at startup, mirroring `dock_layouts.rs`'s own lazy `create_dir_all`).
No rollback concern beyond deleting the new directory.

## Open Questions

None - storage shape, filename derivation and collision handling, Add/Replace semantics, missing-
context/namespace/object handling, the Settings section split, and the keybinding table above are
decided; `keymap::conflicts` is still the authoritative check at implementation time, which does
not change this capability's behavior.
