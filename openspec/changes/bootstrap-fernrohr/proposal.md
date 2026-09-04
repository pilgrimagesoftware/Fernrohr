## Why

Fernrohr has no code yet. Before any Kubernetes feature work is worthwhile, the risky integration
seam needs to exist and be proven: GPUI's foreground event loop driving a `tokio` runtime that owns
`kube-rs`, with live `watcher` streams feeding the UI. This change builds the thinnest end-to-end
slice that exercises that seam - connect to one cluster, show a live-updating Pods table, stream a
pod's logs - plus the app shell (multi-window docked panels), the keyboard-first command system, and
platform config paths. Everything after this is additive.

## What Changes

- New Rust workspace: a GPUI + gpui-component desktop application targeting macOS and Linux.
- App shell: a main window with a gpui-component `Dock` layout; open additional windows; each window
  holds independent panels. Window geometry and open panels persist across restarts.
- The spine: a multi-thread `tokio` runtime on background threads, bridged to GPUI's foreground
  executor through bounded channels. `kube-rs` `watcher` deltas are mirrored into app-scoped GPUI
  entities (per cluster + kind); window-scoped views observe them.
- Cluster connection: read `$KUBECONFIG` then `~/.kube/config` (read-only, never written), list
  contexts, connect one selected context, run API discovery. No tunnels in this change.
- Resource browser: one panel type showing a live Pods table backed by a `watcher`, with namespace
  selector, text filter, and column sort. Watchers are reference-counted per (cluster, kind) and stop
  when the last panel using them closes.
- Pod logs: stream a selected pod's logs with follow and a container picker.
- Command system: a central command registry where every action has an id, title, default keybinding,
  and context predicate. A fuzzy command palette and a user-editable global `keymap.toml` both read
  from it. GPUI `actions!` / `KeyBinding` / `KeyContext` under the hood.
- Platform paths: a single `paths.rs` resolving state, preferences, and cache directories via the
  `dirs` crate (reverse-domain id on macOS, `fernrohr` on Linux). Per-concern TOML files with typed
  serde structs and `load()` / `save()`; first run writes defaults; a parse failure keeps the file and
  falls back to defaults.
- i18n scaffolding so user-facing strings are externalized from the start.

Non-goals for this change: SSH tunnels / port-forwarding, Prometheus / metrics, resource kinds beyond
Pods, YAML editing, exec / shell, Helm, RBAC awareness, per-workspace keymap overrides.

## Capabilities

### New Capabilities

- `app-shell`: main window, `Dock`-based panel layout, multi-window support, and persistence of window
  geometry and open panels to the platform state directory.
- `cluster-connection`: kubeconfig context discovery (read-only), connecting a single context, and
  running Kubernetes API discovery for that connection.
- `resource-browser`: a watch-backed live resource table (Pods) with namespace scoping, text
  filtering, and column sorting, over reference-counted watchers.
- `pod-logs`: streaming a pod's container logs with follow mode and container selection.
- `command-system`: the command registry, fuzzy command palette, and user-editable global keymap
  loaded from `keymap.toml`.

### Modified Capabilities

None - this is the first change.

## Impact

- New: Cargo workspace, GPUI + gpui-component dependency stack pinned to compatible git revisions,
  `kube` / `kube-runtime` / `k8s-openapi`, `tokio`, `dirs`, `serde` / `toml`, an i18n crate.
- New modules: app entry + runtime bridge, `paths.rs` + config loaders, cluster session + watcher
  registry, dock/panel shell, Pods resource view, log stream view, command registry + palette + keymap.
- Platform: writes under `dirs::data_dir()` / `dirs::preference_dir()` / `dirs::cache_dir()`; reads
  kubeconfig; opens outbound HTTPS to the selected cluster's API server.
- Risk: GPUI's Linux (Wayland/X11) backend is less mature than macOS - Linux is a first-class test
  target for this change, not a follow-up.
