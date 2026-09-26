# Architecture

Fernrohr is a native Kubernetes cluster GUI in Rust. Broadly a FreeLens/OpenLens equivalent with
k9s-inspired keyboard-first workflows and multi-window / multi-panel support.

This document mirrors the `context:` block in `openspec/config.yaml`, which OpenSpec feeds into
every proposal and apply operation as shared background. Keep the two in sync when either changes.

## Tech stack

- Rust, GPUI (Zed's UI framework) + gpui-component (Longbridge component lib: Dock, Table, Sidebar,
  TitleBar, Input, etc.), consumed as a single dependency via `gpui-kit`, which bundles the matched
  Longbridge fork stack. `gpui-component` depends on a Longbridge fork republished as `gpui-pre`
  (not upstream `zed-industries/gpui`), so depending on raw `gpui` + `gpui-component` together gives
  two incompatible `gpui` types.
- kube-rs (`kube`, `kube-runtime`, `k8s-openapi`) for the Kubernetes client; `watcher` + reflector
  `Store` for live caches; `discovery` for CRDs.
- tokio multi-thread runtime on background threads, bridged to GPUI's foreground executor via
  bounded channels.
- `dirs` crate for platform paths; per-concern TOML config files; `keyring` crate for secrets.

Targets: macOS and Linux. GPUI's Linux backend is less mature than macOS - a known risk, and a
first-class target rather than a follow-up (see `bootstrap-fernrohr`'s task 9.1).

## Architecture decisions

- **The spine**: a `tokio` runtime owns `kube-rs`; `watcher` deltas are mirrored into app-scoped
  GPUI `Entities` (`Vec` + `HashMap<Uid>` per cluster+kind); window-scoped `Views` observe those
  entities. Bounded channels with coalescing for backpressure during relist bursts.
- **Watcher lifecycle** is reference-counted per `(cluster, kind)`: the last panel closed across
  all windows stops the watcher.
- **App-global state**: `ClusterRegistry`, `ClusterSession[ctx]` (`kube::Client`, watchers, `Store`
  cache). Windows own a `Dock` layout; each panel is `(ClusterSession` ref, `ResourceView`: kind +
  namespace + filter + sort + selection`)`.
- **Multi-cluster side-by-side panels** in one dock is an explicit differentiator vs. Lens.
- **Keyboard-first** via GPUI's `actions!`/`KeyBinding`/`KeyContext`. A central command registry:
  every action has an id, title, default keybind, and context predicate. Menus, palette, and keymap
  all read from it. One global `keymap.toml` for now; per-workspace overrides later.
- **Tunnels**: SSH tunnels and Kubernetes port-forwards share one `ManagedForward` abstraction
  (state machine, refcount, local-bind allocation, health, UI row). A kube context binds to 0 or 1
  tunnel; many contexts may share one. A bound cluster connects only when its tunnel is `Up`; a
  tunnel flap pauses (not destroys) that cluster's watchers. The same pause/resume path handles
  exec-auth (`aws eks get-token` / `gcloud`) 401 re-auth. SSH implementation: shell out to `ssh -L`
  for MVP, swap to `russh` later behind the trait. URL rewrite to `127.0.0.1:PORT` with
  `kube::Config.tls_server_name` set to the real apiserver host so cert validation still passes.
  Tracked in the `tunnel-subsystem` OpenSpec change (depends on `bootstrap-fernrohr`).
- **Prometheus**: default path queries through the kube apiserver service proxy
  (`/api/v1/namespaces/{ns}/services/{svc}:{port}/proxy/api/v1/query_range`), so it piggybacks the
  existing (possibly tunneled) kube client with no extra config. A provider abstraction
  (lens/helm/helm-14/operator/stacklight/openshift) - each finds its Prometheus `Service` by label
  selector and carries its own PromQL templates. A `directUrl` mode (OpenShift / kube-rbac-proxy,
  optional bearer token) is opt-in and needs its own `ManagedForward`.
- **Persistence** (pattern from `pilgrimagesoftware/dtrpg-app.rs`): the `dirs` crate, one central
  `paths.rs`. macOS directory name is the reverse-domain bundle id, Linux is `fernrohr`.
  `dirs::data_dir()/<id>/` holds state (`window_state.toml`, `workspace.toml`, `tunnels.toml`
  non-secret fields). `dirs::preference_dir()/<id>/` holds `ui.toml` and `keymap.toml`.
  `dirs::cache_dir()/<id>/` holds the discovery cache and metric buffers. Each file has a typed
  serde struct with `load()`/`save()`: first run writes defaults; a parse failure keeps the file
  untouched and falls back to defaults. Secrets (SSH passphrases, passwords) go in the OS keychain
  via `keyring` (Apple-native on macOS, Secret Service on Linux; falls back to a prompt when no
  daemon is available). kubeconfig is read-only, never owned (`$KUBECONFIG` then `~/.kube/config`).

## Conventions

- American English in code, comments, and docs.
- Keep functions under ~50 lines, files under ~700 lines.
- Tests: `cargo test`, assertions that verify behavior, no live-cluster dependency (mock the kube
  client or use recorded fixtures).
- i18n from the start for user-facing strings.

## Repository layout

Fernrohr is a meta-repository: `App` is the `pilgrimagesoftware/Fernrohr-App` submodule holding the
actual Rust workspace. `openspec/changes/` holds the OpenSpec changes driving implementation; see
`openspec/changes/bootstrap-fernrohr/` for the in-progress end-to-end slice (app shell, cluster
connection, resource browser, pod logs, command system) and `openspec/changes/tunnel-subsystem/`
for the not-yet-started tunnels/port-forward work described above.
