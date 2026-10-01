# Design

## Context

The automatic restore already in place (`app/src/util/shell/persist.rs`,
`app/src/config/workspace.rs`, `app/src/config/dock_layouts.rs`) is the foundation this builds on:

- `WorkspaceConfig { windows: Vec<WindowLayout> }` persists to `state_dir()/workspace.toml`
  (TOML). `WindowLayout` holds window bounds (`width`, `height`, `x`, `y`), the window's
  `contexts: Vec<String>`, a `resource_panel_width`, and a `panels: Vec<PanelDescriptor>` field
  that exists but is currently always saved empty - panel content restoration today actually comes
  from the second file below.
- `state_dir()/dock-layouts.json` (JSON, via `serde_json`) stores `DockLayouts { layouts:
  HashMap<String, DockAreaState> }`, keyed by `context_lifecycle::dock_layout_key(contexts)`
  (single context = bare name; multiple = sorted names joined with `\u{1f}`). `DockAreaState` is
  `gpui_kit`'s own dock-dump type: it already carries the split tree, pane sizes, tab order, active
  tab, zoom, and each panel's opaque `data` field, which `restored_panel_keys()`
  (`app/src/ui/nav.rs` area) deserializes into a `PanelKey { target: NavTarget, context_name,
  namespaces }` - this `data` field is where a panel's kind/object/namespace/context already live
  today, per panel.
- Both files are written from one `cx.on_app_quit()` callback; the dock file is also kept live via
  `SavedDockLayouts`, updated on every `DockEvent::LayoutChanged`.
- Commands are `Command { id, title, default_binding, context: Option<&str>, action, menu:
  Option<MenuSlot> }` in a `CommandRegistry` global (`app/src/command.rs`); `keymap.rs` derives
  `keymap.toml` defaults from the registry, and `keymap::conflicts` already checks for colliding
  bindings. `ui/menu.rs` builds each menu, including `MenuSlot::Window`, from `registry.for_menu`.
  `ui/picker/` (`ClusterPicker`) is the existing keyboard-first, searchable-list reference: it
  wraps `gpui_kit::component::command::{Command, CommandState}`, distinguishes a keyboard-driven
  highlight from mouse hover via `window.last_input_was_keyboard()`, and a second click/Enter (not
  hover) confirms a row.
- `SecretValue` (`app/src/k8s/resource/secret_value.rs`) has no `Serialize` impl by construction;
  nothing this change does can put a revealed value into a saved layout, because there is no path
  for it to reach serde at all.

See proposal.md for motivation.

## Goals / Non-Goals

**Goals:**
- Add a second, named, user-managed layout store that sits next to the automatic one without
  changing its behavior.
- Reuse the existing dock-dump (`DockAreaState`) and panel-key machinery rather than inventing a
  parallel representation of "what a panel shows".
- Make every new action a registered command: palette entry, default binding, Window-menu item.
- Never lose a panel on restore, even when its context, namespace, or object is gone.

**Non-Goals:**
- Per-workspace keymap overrides (`architecture.md` already defers this; out of scope here).
- Syncing saved layouts across machines or users.
- Saving/restoring anything from the Resource panel's content beyond its width and visibility
  (its own scroll position, filter text, etc. are out of scope - the same granularity the
  automatic restore already stops at).
- Changing `object-detail`'s Secret-redaction requirement; this change only depends on it.

## Decisions

### D1: A new capability, not a modified `app-shell`
`app-shell`'s requirement ("persist, per window... and restore them on the next launch") describes
the *automatic, implicit, singular* restore and stays true unchanged. Named saves are a distinct,
user-initiated, multi-slot concept layered on top, so they get their own capability
(`saved-panel-layouts`) rather than reworded `app-shell` text. This also avoids a merge collision
with the in-flight `window-title-and-menu` change, which separately modifies `app-shell` and
`application-menu`.

### D2: Storage format and location
A new `state_dir()/saved-layouts.json` (JSON, not TOML), because its payload embeds the same
`DockAreaState` shape `dock-layouts.json` already stores, and that type is already serialized as
JSON today - reusing the format avoids a second, parallel serialization path for the same data.

```rust
struct SavedLayouts {
    version: u32,            // schema version; bump on breaking shape changes
    layouts: Vec<SavedLayout>,
}

struct SavedLayout {
    name: String,            // unique; the picker's key and display text
    created_at: String,      // RFC 3339, for picker sort/display
    updated_at: String,      // bumped on overwrite and rename
    contexts: Vec<String>,   // same shape as WindowLayout.contexts
    dock: DockAreaState,     // same type dock-layouts.json already stores
    resource_panel_width: Option<f32>,
    resource_panel_visible: bool,
    window_width: f32,
    window_height: f32,
}
```
`load()`/`save()` follow the existing per-file convention in `persist.rs`: a parse failure leaves
the file untouched and falls back to an empty `SavedLayouts` (consistent with how `workspace.toml`
and `dock-layouts.json` already fail safe) rather than a hard startup error - matching this
change's "unreadable file starts with none, doesn't fail, doesn't clobber" requirement.
Unlike the automatic files, this one is written only on explicit user action (save, rename,
delete, overwrite), not on every dock change, so there is no live-update subscription to wire.

**Alternative considered:** fold saved layouts into `workspace.toml` as a new field. Rejected:
that file's `WindowLayout` is inherently per-*current-window*, singular, and already rewritten
wholesale on every quit; mixing in a named, multi-entry, user-curated list would make an accidental
loss (a bad `workspace.toml` write) take the user's saved work down with it.

### D3: Content key reuse
A saved layout's `dock: DockAreaState` carries panel `data` in the same shape
`restored_panel_keys()` already decodes into `PanelKey`/`NavTarget`. Restoring a saved layout
reuses that exact decode path, so a panel's kind/object/namespace/context round-trips through the
same code the automatic restore already exercises and already has tests for. No new "what does
this panel show" representation is introduced.

### D4: Restoring a missing context, namespace, or object
Decoding a `PanelKey` happens before a context is known to be connected - the existing automatic
restore already tolerates this (a context reconnects lazily). This change makes the same lazy-
connect path explicit and adds a terminal case it doesn't yet need: a context absent from the
*current* kubeconfig entirely. Restore classifies each panel's context into:
- **Connected already** -> build the panel immediately.
- **Known, not connected** -> connect on demand (same path `ClusterConnection` already uses from
  the cluster picker), panel shows its normal "connecting" state meanwhile.
- **Unknown to the current kubeconfig** -> placeholder panel, same slot in the dock tree, offering
  to pick a replacement context (reuses the cluster-picker affordance in miniature) rather than
  silently closing that pane.

A namespace or object that is gone is a cluster-level `404` surfaced only once the panel actually
queries - handled the same way `pod-detail`'s existing "pod no longer exists" requirement already
does for pod detail panels. This change applies that same pattern (placeholder + retry, not an
error dialog) to the other panel kinds a saved layout can restore.

### D5: Restore-in-place vs. restore-in-new-window are two commands, not one dialog option
Keyboard-first: a user holding a key through a picker to confirm shouldn't hit a modal fork
afterward asking which window. Two registered commands, scoped to the picker's `KeyContext`
(`SavedLayoutsPicker`), each with its own binding, mirror the existing pattern of panel-scoped
commands (`object-detail`'s per-tab, per-action bindings):

| id | title | default binding | context |
|---|---|---|---|
| `layouts.save` | Save Panel Layout… | `cmd-shift-s` | `Workspace` (not available from the empty cluster picker) |
| `layouts.manage` | Saved Layouts… | `cmd-shift-o` | none (available even from the cluster picker, so a fresh window can jump straight to a saved layout) |
| `saved_layouts.restore_in_window` | Restore Here | `enter` | `SavedLayoutsPicker` |
| `saved_layouts.restore_in_new_window` | Restore in New Window | `cmd-enter` | `SavedLayoutsPicker` |
| `saved_layouts.rename_selected` | Rename | `r` | `SavedLayoutsPicker` |
| `saved_layouts.delete_selected` | Delete | `backspace` | `SavedLayoutsPicker` |

None of `cmd-shift-s`, `cmd-shift-o`, or the scoped single-letter bindings collide with the
bindings already registered in `app/src/command.rs`, `ui/nav.rs`, `ui/menu.rs`,
`util/shell/app.rs`, `ui/tunnels/list/command.rs`, or `ui/settings.rs` (checked by reading each
file's `default_binding` at proposal time); `keymap::conflicts` is still the authoritative check
at implementation time, and a task below runs it. Delete asks for inline confirmation (a second
press, or a confirm row) rather than a nested modal, consistent with "dialogs are keyboard-
operable" rather than stacking two.

Restoring in the current window replaces that window's dock, Resource panel state, and bounds with
the saved ones (the window moves/resizes to match - treated as "I asked for *that* layout", not a
content-only swap). Restoring into a new window opens one at the saved bounds, offset the same way
`layouts.save`'s own `NEW_WINDOW_DEFAULT_BINDING` path already cascades new windows, so two
restores of the same layout don't land exactly on top of each other.

### D6: Secret safety is inherited, not re-implemented
Because `SavedLayout.dock: DockAreaState` is produced by the same dock-dump path
`dock-layouts.json` already uses, and `SecretValue` has no `Serialize` impl, a revealed value
cannot reach `saved-layouts.json` through any serialization path - there is nothing extra to
redact. A test (mirroring `object-detail`'s own "revealed value stays out of saved state" test)
asserts a `format!("{:?}")` of a saved layout containing a revealed value has no fixture value in
it, as a structural regression guard rather than a behavioral one.

## Risks / Trade-offs

- [A saved layout's dock JSON references panel kinds the current build doesn't know about, from a
  newer version that wrote the file] -> `PanelDescriptor`/`PanelKey` decoding already has an
  `Unknown`/skip path for this (the automatic restore relies on it too); a saved layout with such a
  panel restores every other panel and shows that one slot as "not restorable in this version"
  rather than failing the whole restore.
- [Window size/position in a saved layout no longer fits any connected display (laptop vs. external
  monitor)] -> reuse `layout.rs`'s existing `window_bounds()` clamping/centering fallback, which
  already handles a missing or stale position for the automatic restore.
- [A user deletes a saved layout by mistake] -> inline confirmation on Delete (D5); no undo is
  added in this change, matching the lack of undo elsewhere in the picker family today.
- [Saving very often (once per task switch) grows `saved-layouts.json` unbounded] -> no retention
  limit in this change; left as a follow-up if it proves to matter in practice, consistent with
  not inventing a requirement the proposal didn't ask for.

## Migration Plan

Purely additive: a new file, new commands, new menu entries. No existing file format, command id,
or keybinding changes. First run with no `saved-layouts.json` behaves exactly as today until the
user saves a layout for the first time (file is created then, not eagerly at startup, mirroring
`workspace.toml`'s own first-run-creates-defaults behavior read loosely - here "defaults" is just
an empty list). No rollback concern beyond deleting the new file.

## Open Questions

None - the save/restore split, storage format, and missing-reference handling above are decided;
the keybinding table is a proposed default subject to confirmation against `keymap::conflicts` at
implementation time, which does not change the capability's behavior.
