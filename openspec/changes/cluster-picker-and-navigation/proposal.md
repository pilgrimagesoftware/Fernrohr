# Proposal

## Why

The app currently has no front door: `shell::open_window` hardcodes a fixed
two-panel dock (Pods + Logs) with no cluster selection and no way to switch
resource kinds. `cluster-connection`'s "connect to one user-selected context"
requirement was never given a UI, and `resource-browser` only ever shows
Pods. A user cannot actually launch the app, pick a cluster, and look around
- which is the app's basic reason to exist.

## What Changes

- Add a startup screen shown when a window has no restored panels: lists
  kubeconfig contexts (via the existing `cluster::kubeconfig` module),
  lets the user pick one, and drives `cluster::connection` to connect.
- On successful connection, replace the picker with the window's panel
  workspace, landing on a default resource view for that cluster instead of
  a hardcoded Pods+Logs split.
- Add resource-kind navigation (a sidebar) so an open panel can switch which
  resource kind it displays, backed by the discovery data `cluster-connection`
  already exposes, instead of one kind being wired in at panel-construction
  time.
- Surface a connection failure (bad context, unreachable cluster) on the
  picker itself rather than failing silently into a blank window.
- Replace the fixed Pods/Logs sidebar with a Resource panel that lists every
  kind a cluster's API discovery reports, including CRDs, and opens a
  dockable panel for a selected kind (double-click or context menu).
- Give resource panels a title bar showing kind, cluster name (when more
  than one connection is open in the window), a namespace picker for
  namespaced kinds, a controls menu, and a close button.
- Anchor the Resource panel to either edge of the window (user preference,
  movable at runtime, collapsible), and make dockable panels focus-aware
  (one focused panel at a time, visually distinguished, receives panel
  keyboard shortcuts) and individually maximizable (at most one at a time,
  excluding the Resource panel's space). Panel move/tile/split/persist ride
  on `gpui-kit`'s existing `DockArea` (drag-to-dock, splits, zoom, layout
  dump/load) rather than new layout-engine work.
- Restore a cluster's last saved layout on connect when one exists, falling
  back to the full Resource panel (not a hardcoded split) when it doesn't.
- Wire the app's existing but unwired `UiConfig`/`Theme` into startup (applied
  once before any window exists, re-applied per window, and following a
  mid-session OS appearance change when set to `System`), and build the picker
  and workspace sidebar from the app's own design system — the `Command` widget
  the command palette already uses, gpui-component's `Sidebar` — rather than
  bespoke `div`s.
- Make visual space in the Resource panel for a cluster dropdown, shown once
  a window has more than one cluster connection open. Actually opening a
  *second* connection within an already-connected window is out of scope
  for this change (see Non-Goals in `design.md`) - the dropdown and its
  single-connection empty state are what land now.
- **BREAKING**: `shell::open_window`'s hardcoded panel construction is
  removed; a window with no saved layout now opens the picker, not
  Pods+Logs.

## Capabilities

### New Capabilities
- `cluster-picker`: the startup/context-selection screen and its connect flow.

### Modified Capabilities
- `app-shell`: what a window shows before any panel is open or restored
  (the picker, or a per-cluster saved layout), Resource panel placement,
  panel focus, and panel maximize.
- `resource-browser`: the Resource panel lists full cluster discovery
  (including CRDs) and opens dockable panels with a full title bar, instead
  of a fixed Pods/Logs sidebar.

## Impact

- `app/src/shell.rs`: `open_window` no longer hardcodes panel construction;
  branches on whether the restored layout has panels, and on which cluster
  it's restoring a layout for.
- `app/src/cluster/`: `kubeconfig.rs` (list contexts) and `connection.rs`
  (connect) get a UI-facing entry point; `discovery.rs` output feeds the
  Resource panel's full kind list instead of a fixed sidebar.
- New UI module for the picker view and the Resource panel (kind list,
  cluster dropdown, anchor/collapse).
- New `app/src/theme.rs` plus startup wiring in `main.rs`: `ui.toml` is loaded
  and the theme applied once before the first window opens, then re-applied
  per window.
- `resource_index.rs` / `pods.rs`: panel construction decouples from "always
  Pods" to "whatever kind is selected"; panel title bars gain cluster name,
  namespace picker, controls menu.
- `app/src/dock` (or wherever `DockArea` is wired today): panel focus
  tracking and maximize/zoom wiring on top of `gpui-kit`'s existing dock
  primitives.
