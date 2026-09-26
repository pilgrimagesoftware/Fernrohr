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
- Existing `Pods`/`Logs` panels become reachable via navigation instead of being hardcoded
  into every window.
- Each window is independent: opening a new window (the existing `NewWindow`
  action) shows that window's own picker, and it can connect to a different
  cluster than any other open window, fully side-by-side. Two windows
  connected to the same context share one `ClusterSession`/watch; two windows
  connected to different contexts get independent ones.

**Non-Goals:**
- Two clusters shown side-by-side *inside one window's dock* (e.g. a
  split with a Pods panel from cluster A next to a Pods panel from cluster B
  in the same window) — that's a dock/panel-construction feature, not a
  connection-model one, and isn't needed to satisfy this change's specs.
  Multiple *windows* each connected to their own cluster (this change's
  actual goal, above) already falls out of the per-window picker plus the
  context-keyed `ClusterRegistry`.
- Generic per-kind resource tables for arbitrary discovered kinds — navigation this change
  adds only switches among kinds the app already implements (Pods, Logs).
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

**Navigation is a fixed sidebar of the app's implemented views, not a discovery-driven menu.**
`resource-browser`'s "reflects that cluster's discovery" scenario is satisfied by filtering
this fixed list against discovery output (e.g. hiding Logs if Pods aren't reachable is out of
scope; the realistic filter today is close to a no-op since discovery isn't yet used to gate
anything). Building a fully discovery-driven, arbitrary-kind navigation menu is deferred per
Non-Goals; the sidebar is a small, explicit `NavTarget` enum today (`Pods`, `Logs`), not a
generic list keyed by API resource.

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
