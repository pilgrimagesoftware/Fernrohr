## Context

Greenfield repository. See proposal.md - Why. The whole point of this change is to prove the GPUI +
`tokio` + `kube-rs` integration seam, so the design is mostly about that seam and the state model that
hangs off it. GPUI and gpui-component are fast-moving and version-coupled; `kube-rs` is
`tokio`-native and GPUI runs its own single-threaded foreground executor. macOS and Linux are both
first-class targets.

## Goals / Non-Goals

**Goals:**

- One documented, reusable pattern for moving data from `tokio`/`kube-rs` onto the GPUI foreground
  executor and into observable UI state.
- A state model (app-scoped resource stores, window-scoped views) that later resource kinds, tunnels,
  and metrics slot into without rework.
- Watch lifecycle that is correct under multi-window, multi-panel use from day one.
- Config/persistence layout that the tunnel and metrics changes extend rather than replace.

**Non-Goals:**

- Any abstraction for tunnels, port-forwarding, or metrics - those get their own changes. This change
  must not add trait seams "for later" that aren't exercised now.
- Generic dynamic/CRD resource rendering. Pods are typed via `k8s-openapi`.
- A reusable virtualized table framework beyond what gpui-component's `Table` gives.

## Decisions

### D1: Dedicated `tokio` runtime owned by the app, bridged with bounded channels

A multi-thread `tokio` runtime is built at startup and its handle stored in a GPUI global. `kube-rs`
work (`watcher`, `log_stream`, discovery) runs as `tokio` tasks. Each task forwards results over a
bounded `mpsc` channel; a GPUI foreground task drains the channel and applies updates to entities.

- _Why not drive `kube-rs` on GPUI's executor?_ `kube`/`hyper` require a `tokio` reactor;
  GPUI's executor is not one.
- _Why bounded, not unbounded?_ A relist on a large cluster emits thousands of events in a burst. An
  unbounded channel turns that into unbounded memory and a foreground-thread stall. Bounded + a
  coalescing drain (fold consecutive updates to the same UID before touching the entity) keeps paint
  responsive. Backpressure on the producer during a burst is acceptable.
- _Alternative considered:_ `smol`/`async-std` to match GPUI's async style - rejected, `kube-rs`
  ecosystem assumes `tokio`.

### D2: Mirror watch deltas into app-scoped entities; views observe

For each `(cluster, kind)` there is one app-scoped entity holding an index: `Vec<Arc<K>>` plus
`HashMap<Uid, usize>`. The drain task applies `Applied`/`Deleted` events to this index. Views hold a
handle to the entity and re-render on change via GPUI's observation.

- _Why mirror instead of reading `kube-runtime`'s reflector `Store` from views?_ Mirroring gives
  GPUI change notifications for free and lets the drain task coalesce and prune fields. Reading the
  `Store` from views means each view polls and diffs.
- _Field pruning:_ list rows need a small subset (`metadata`, a few `status` fields). For this
  change Pods are small enough to keep whole; the index type is defined so a later change can swap in
  a projected struct without touching views.
- _Alternative considered:_ one global object store keyed by GVK+UID - rejected as premature; the
  per-`(cluster, kind)` entity is simpler and matches the watch lifecycle unit.

### D3: Watch registry with reference counting

A `WatchRegistry` on the `ClusterSession` maps `kind -> (entity, task handle, refcount)`. Opening a
panel calls `subscribe(kind)` (starts the task on 0→1); closing calls `unsubscribe(kind)` (aborts the
task and drops the entity on 1→0). The count spans all windows because sessions are app-scoped.

- _Why here and not per window?_ Two windows showing Pods for the same cluster must share one
  stream. The refcount unit is the same as the entity unit from D2.
- _Reconnect:_ `kube_runtime::watcher` already handles relist/backoff internally; the task just keeps
  consuming the stream. A terminal error restarts the task with backoff.

### D4: Command registry as the single source for palette + keymap

A `CommandRegistry` holds `Command { id, title, default_binding, context }`. At startup it is
populated, `keymap.toml` is layered on top, and the result is installed as GPUI `KeyBinding`s scoped
by `KeyContext`. The palette lists `Command`s filtered by the active context stack.

- _Why wrap GPUI's action system rather than use it directly?_ GPUI actions give dispatch and
  key contexts but no title metadata, no user-facing list, no external keymap file. The registry adds
  exactly those and delegates the rest to GPUI.
- _Scope:_ global `keymap.toml` only. Per-workspace overrides are a later change; the loader takes a
  single layer now.

### D5: Persistence via a central `paths.rs` and per-concern TOML (pattern from dtrpg-app.rs)

`paths.rs` resolves three roots with the `dirs` crate: `data_dir()` (state:
`window_state.toml`, `workspace.toml`), `preference_dir()` (`ui.toml`, `keymap.toml`),
`cache_dir()` (discovery cache). Directory name is a reverse-domain id on macOS, `fernrohr`
elsewhere. Each file has a typed `*File` serde struct with `load()`/`save()`: first run writes
resolved defaults; a parse failure logs, keeps the file untouched, and uses defaults.

- _Why `dirs` not `directories`?_ Matches the reference implementation; `dirs::preference_dir()`
  gives the macOS `~/Library/Preferences` split that `directories` collapses.
- _Why per-concern files not one config?_ Independent load/save, smaller blast radius on a corrupt
  file, and the tunnel change can add `tunnels.toml` without merging into a monolith.
- kubeconfig is read via `kube`'s own loader (`$KUBECONFIG` then `~/.kube/config`) and never written.

### D6: Workspace/window state model

`workspace.toml` stores, per window: geometry and a list of panel descriptors
`{ kind: "pods" | "logs", cluster_context, namespace, filter, sort }`. On launch the app rebuilds
windows and panels from this; panels re-`subscribe` their watches. Unknown panel `kind`s are skipped
with a log line (forward-compat for later panel types).

## Risks / Trade-offs

- **GPUI Linux (Wayland/X11) backend is younger than macOS** → Linux is in the CI matrix and in the
  task list's manual-verification step for this change; do not defer Linux validation.
- **GPUI / gpui-component API churn** → pin both to specific git revisions in `Cargo.toml`; a single
  `deps` task owns bumps. Accept periodic breakage on upgrade.
- **Foreground-thread stalls from event bursts** → bounded channel + coalescing drain (D1); a manual
  test against a namespace with hundreds of Pods is in the task list.
- **exec credential plugins block on first call** → run the client build and first request on the
  `tokio` runtime, never on the foreground executor; surface a "connecting" state.
- **Coalescing drops intermediate states** → acceptable for a table view; log follow (pod-logs) does
  not go through the coalescing path, it streams line-by-line on its own channel.
- **Over-abstracting for future changes** → explicit non-goal; review the diff for trait seams with a
  single implementation and delete them.

## Open Questions

- Exact gpui-component revision to pin and whether its `Table` handles the row counts we expect
  without a custom virtualization pass - resolved during the first task, does not affect specs.
- Whether `window_state.toml` and `workspace.toml` should be one file - deferred; the loader is
  per-file either way.
