# Design

## Context

`cluster::kubeconfig::list_context_names` already exists and is unwired (`#[allow(dead_code)]`).
`ClusterConnection::connect(cx)` takes no context argument at all — it always resolves
`Config::infer()`, i.e. whatever the kubeconfig's own `current-context` is. `ClusterSession`
is a single GPUI `Global`, lazily created on first `subscribe_pods`/`connection` call, holding
exactly one `ClusterConnection` and one shared `PodsTable` for the whole app. `shell::open_window`
hardcodes a `PodsPanel` + `LogsPanel` split on every window, regardless of `WorkspaceConfig`.
`PanelDescriptor::{Pods, Logs}` already carry a `cluster_context: String` field, but
`shell::restorable_panels` only filters out `Unknown` variants — nothing yet reconstructs a
panel from a descriptor, so persisted panels are inert data today.

So "pick a cluster" cannot be added as pure UI: `ClusterConnection::connect` and
`ClusterSession` need to accept which context to use instead of assuming there is exactly one.

## Goals / Non-Goals

**Goals:**
- A window with no restored panels shows the picker; selecting a context connects to
  that specific context, not whatever `current-context` happens to be.
- `ClusterSession` becomes keyed by context name (a small registry), so a second window
  can pick a different cluster later without redesigning this change's plumbing again.
- The Resource panel lists a connected cluster's full API discovery (including CRDs),
  not a fixed `Pods`/`Logs` pair, and opens a dockable panel for whichever kind is
  selected instead of kinds being hardcoded into every window.
- A cluster's last saved layout restores on connect when one exists; otherwise the
  Resource panel (full discovery) is shown instead of a hardcoded split.
- Each window is independent: opening a new window (the existing `NewWindow`
  action) shows that window's own picker, and it can connect to a different
  cluster than any other open window, fully side-by-side. Two windows
  connected to the same context share one `ClusterSession`/watch; two windows
  connected to different contexts get independent ones.

**Non-Goals:**
- Opening a *second cluster connection within an already-connected window* — the
  Resource panel makes visual room for it (the cluster dropdown, shown once more than
  one connection exists), but the flow to actually add that second connection is
  follow-up work. Multiple *windows* each connected to their own cluster (this change's
  actual goal, above) already falls out of the per-window picker plus the
  context-keyed `ClusterRegistry`.
- Free-floating, arbitrarily-positioned panels with edge-to-edge snapping between them
  and a modifier-key snap override — dropped from this change's scope. `gpui-kit`'s
  `DockArea` panels live in a split/tab tree, not at arbitrary positions; "snap" is
  satisfied by its existing drag-to-dock-into-a-split behavior, not a new floating
  layout engine.
- Panel minimize (collapse to a title-bar strip or similar) — dropped from this change's
  scope. `DockArea` has zoom (this change's "maximize"), not minimize; revisit only if a
  concrete need for it shows up later.
- Full panel-layout restoration from `PanelDescriptor` (split arrangement etc.) — this change
  only needs enough restoration to know a window has panels and should skip the picker;
  reconstructing the exact prior split is separate work `restorable_panels`'s doc comment
  already flags as pending.
- Tunnel/exec-auth UI (already covered by the tunnel-subsystem change).

## Decisions

**`ClusterConnection::connect` takes an explicit context name.**
Change its signature to `connect(cx, context_name: Option<String>)`. `Some(name)` resolves
via `Kubeconfig::read()` + `Config::from_custom_kubeconfig(.., &KubeConfigOptions { context: Some(name), .. })`;
`None` keeps today's `Config::infer()` path (in-cluster config, or the kubeconfig's own
current-context) for any caller that doesn't care yet. Alternative considered: keep
`connect()` argument-free and add a second `connect_to(name)` — rejected, since every real
caller after this change knows which context it wants, and two entry points would only
invite the old one to keep being called by mistake.

**`ClusterSession` becomes a registry keyed by context name.**
Replace the single `Global` struct with `ClusterRegistry: Global` holding
`HashMap<String, ClusterSession>`, and change `ClusterSession::connection`/`subscribe_pods`/etc.
to take a `context_name: &str` alongside `cx`, looking up or lazily creating that context's
entry. This matches the `ClusterRegistry`/`ClusterSession[ctx]` design already recorded in
`openspec/config.yaml`; today's single-`Global` shape was a bootstrap shortcut, not the
intended end state. Alternative considered: keep one global session and just reconnect it
when the user picks a different context — rejected, because it would throw away an existing
connection's watches every time the picker is used again from a second window, contradicting
the multi-cluster differentiator already decided for the project.

**The picker is a plain view, not a panel.**
It renders in place of `DockArea` inside `MainWindow`, not as a dockable panel — the
picker's whole point is that there is no session (and so nothing to dock into) yet.
`MainWindow` gains a simple enum (`Picker` vs `Workspace(Entity<DockArea>)`) and swaps on
successful connect.

**The Resource panel is a discovery-driven list, not a fixed sidebar.**
Superseding the original "fixed `NavTarget` enum" plan: the Resource panel renders
directly from `discovery.rs`'s output for the panel's active cluster connection, so CRDs
show up without any per-kind registration. Opening a kind still goes through a small
`NavTarget`-shaped construction step internally (something has to map a discovered
`GroupVersionKind` to a concrete panel type, and only `Pods`/`Logs` have one today) but
the list the user picks from is the cluster's real discovery, not that internal enum.
Kinds without an implemented panel type show in the list but open a placeholder rather
than being hidden, so the list stays an honest reflection of discovery.

**Resource panel anchor and collapse state live per-window, per-preference.**
The edge (left/right) is a user preference with a sensible default, read on window open;
moving it at runtime changes that window's placement only, not the stored preference,
matching the notes' "moved... as the user chooses during runtime" without silently
overwriting the user's stated default. Collapse state is per-window, ephemeral (not
persisted across restarts) — there's no scenario calling for a remembered collapse state,
and persisting one more piece of window chrome isn't needed to satisfy the spec above.

**Panel focus is tracked via GPUI's existing focus system, not a custom notion.**
`DockArea`/`Panel` already participate in GPUI's `FocusHandle` model; "focused panel" in
the spec above means whichever panel's `FocusHandle` currently has window focus. The
"visually distinguished" requirement is a `DockSkin`/`PanelStyle` concern (a border or
title-bar treatment keyed off focus state), not new state to track by hand.

**Panel maximize reuses `DockArea`'s zoom, scoped to exclude the Resource panel.**
`gpui-kit`'s `DockArea` already supports zooming a panel to fill the dock; this change's
"maximize" is that same mechanism, with the Resource panel's region carved out of what
"fill the workspace" means (it lives outside the zoomable center dock, in an edge
placement). No new maximize state machine is needed beyond wiring the existing zoom
action and confirming Resource-panel space is excluded.

## Risks / Trade-offs

- [Risk] Rekeying `ClusterSession` touches `session.rs`, `connection.rs`, and every call site
  in `pods.rs`/`shell.rs` that currently assumes one global session → Mitigation: the field
  and method shapes stay the same per-entry, only the lookup key is added; existing tests in
  `session.rs`/`connection.rs` should need signature updates, not logic rewrites.
- [Risk] `restorable_panels` still doesn't reconstruct panels from descriptors, so "window
  with restored panels skips the picker" can only be satisfied today by treating a non-empty
  descriptor list as "skip the picker," not by actually restoring those panels' content →
  Mitigation: scope tasks so the picker-skip check only depends on `!panels.is_empty()`,
  and leave full reconstruction as follow-up work, consistent with `restorable_panels`'s
  existing doc comment.
- [Risk] Connecting to a context whose credentials are stale/expired surfaces as a generic
  `Failed(String)` on the picker → Mitigation: none needed beyond what `cluster-connection`
  already promises (a descriptive failure); nothing in this change alters that contract.

## Open Questions

- Should the picker remember the last successfully connected context per window and
  preselect it, or always start with nothing selected? Deferred: doesn't change the spec,
  approach, or task breakdown - can be decided during picker implementation.
