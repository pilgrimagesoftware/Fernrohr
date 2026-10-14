# Proposal

## Why

Opening a resource that already has a panel in the same context focuses that panel
(Fernrohr-App#276). That's the right default, but it leaves no way to keep two panels on one
resource: a Deployments list in each of two splits, or a pod's detail on its Overview beside the
same pod's YAML. The user needs a deliberate way to open another instance. Typed label Logs panels
already get one instance per open (`selector-logs-multiple-panels`); this change makes that
possible for every panel kind.

## What Changes

- A new global command, **Open in New Panel** (`panel.open_new_instance`, default `cmd-shift-d`),
  opens another instance of the focused panel: same resource, context and namespaces, as a new
  panel in its own tab with focus. It's in the command palette and the View menu, and it can be
  rebound.
- Every list with rows that open a detail (object lists and the Pods list) gains a row-level route
  that opens the selected or clicked row's resource in a new panel, even when its panel is already
  open:
  - the keyboard: **Open in New Panel** for the list (`shift-enter`), in the palette while the list
    has focus;
  - the mouse: `alt`-click on a row, and an **Open in New Panel** item in the row's context menu.
- Default opens are unchanged: Enter, double-click, links and the Resource panel still focus an
  open panel for the same resource. When several instances are open, they focus the first one
  opened.
- Every panel kind can carry an instance number. The first panel on a resource is instance 0. Each
  further instance gets the lowest number not in use for that resource in the window, and its
  title and tab end in ` (2)`, ` (3)` and so on.
- Saved layouts keep each instance's number, so duplicated panels are restored separately. Layouts
  saved before this change, which hold no number, restore as instance 0.

## Capabilities

### New Capabilities

(none)

### Modified Capabilities

- `app-shell`: adds Open in New Panel for the focused panel, and numbered titles for duplicates.
- `resource-browser`: adds the row-level Open in New Panel route for lists, and adds the item to
  the Pods row context menu.
- `saved-panel-layouts`: adds that each instance of a duplicated panel is saved and restored
  separately.

## Impact

App-only (`App` submodule), no new dependencies, no backend or API changes:

- `app/src/ui/panel/title.rs` - `PanelScope` gains its instance number, and titles and tabs gain
  the suffix.
- `app/src/util/shell/panels.rs`, `open.rs` - `PanelKey` gains the instance; `open_target_in`
  handles a new `OpenMode::NewPanel`, and default opens match any instance.
- `app/src/ui/nav/open_mode.rs`, `app/src/ui/background_rows.rs` - the new mode, and `alt`-click.
- Every panel's save and restore (Pods, object list, events, placeholder, Logs, pod detail, object
  detail, exec) saves and reads `instance`.
- Object list and Pods commands, context menus, and the new global command.
- Docs: `docs/site/latest/windows-panels.md` (Open in New Panel, numbered titles),
  `docs/site/latest/resources.md` (the row routes) and `docs/site/latest/reference/keybindings.md`
  (both commands).
