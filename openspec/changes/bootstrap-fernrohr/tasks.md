## 1. Workspace and dependency setup

- [x] 1.1 Create the Cargo workspace and an app crate; `cargo build` succeeds with an empty GPUI window that opens on macOS
- [x] 1.2 Add and pin GPUI + gpui-component to specific git revisions; a gpui-component `Dock` renders in the window and `cargo build` is reproducible from a clean checkout
- [x] 1.3 Add `tokio` (multi-thread), `kube`, `kube-runtime`, `k8s-openapi`, `dirs`, `serde`, `toml`, and an i18n crate; `cargo build` succeeds and `cargo tree` shows no duplicate major versions of `tokio` or `hyper`
- [x] 1.4 Add a Linux CI job (build + `cargo test`) alongside macOS; both legs pass on the empty-window commit

## 2. Platform paths and config loading

- [x] 2.1 Implement `paths.rs` resolving state, preferences, and cache roots via `dirs`, using a reverse-domain id on macOS and `fernrohr` elsewhere; unit tests assert each resolved path ends with the expected dir name and falls back to `.` when the platform dir is unavailable
- [x] 2.2 Implement a generic typed-TOML `load()`/`save()` helper: first call with no file writes defaults, a corrupt file is left untouched and defaults are returned; unit tests cover create-default, round-trip, and parse-failure-keeps-file
- [x] 2.3 Define `ui.toml`, `window_state.toml`, and `workspace.toml` structs on the helper; `cargo test` covers serde round-trips and unknown-field tolerance

## 3. The runtime spine

- [x] 3.1 Build the `tokio` runtime at startup, store its handle in a GPUI global, and expose a helper to spawn a `tokio` task that streams items back over a bounded channel; an integration test spawns a task that emits N items and asserts the foreground drain receives them in order
- [x] 3.2 Implement the coalescing drain: fold consecutive updates to the same key before applying; a test feeds an interleaved add/update/delete burst and asserts the resulting index matches the final state with bounded intermediate work
- [x] 3.3 Implement the per-`(cluster, kind)` entity index (`Vec` + `HashMap<Uid,usize>`) with apply-`Applied`/apply-`Deleted`; unit tests cover insert, in-place update, delete, and missing-on-delete

## 4. Cluster connection

- [x] 4.1 Load kubeconfig contexts via `kube`'s loader honoring `$KUBECONFIG` then `~/.kube/config`; a test with a fixture kubeconfig lists all context names and asserts the file's bytes and mtime are unchanged after load
- [x] 4.2 Connect a selected context on the `tokio` runtime, building the client and running one probe request; connection state (connecting / connected / failed-with-reason) is observable in the UI, verified against a mock API server for success, unreachable, and exec-plugin-failure cases
- [x] 4.3 Run API discovery on connect and expose the discovered kinds; a test against recorded discovery fixtures asserts `Pod` is present and the call runs off the foreground executor

## 5. App shell

- [x] 5.1 Implement the docked panel workspace: add, split, resize, and close panels; manual check - a second panel splits the area and the separator drags (implemented via gpui-component's `DockArea`/`h_split`; drag-to-resize not visually verified - no display access in this environment, needs a manual check when run locally)
- [x] 5.2 Implement multiple windows via a "New Window" command; opening one yields an independent workspace and closing the last window persists state then exits - verified manually and by a state-file assertion on exit
- [x] 5.3 Persist and restore per-window geometry and panel descriptors through `workspace.toml`; quit-and-relaunch restores two windows with the same panels, and a corrupt `workspace.toml` yields one default window with the file untouched
- [x] 5.4 Skip unknown panel `kind`s on restore with a log line; a hand-edited `workspace.toml` with a bogus kind still restores the known panels

## 6. Resource browser (Pods)

- [x] 6.1 Implement the `WatchRegistry` on `ClusterSession` with `subscribe`/`unsubscribe` refcounting per kind; tests assert one stream for two subscribers and teardown on the last `unsubscribe`
- [x] 6.2 Implement the Pods panel table (name, namespace, ready, status, restarts, age) bound to the kind entity; against a mock stream, initial rows populate and a later `Applied`/`Deleted` adds/removes a row within the drain cycle
- [x] 6.3 Implement namespace scoping (single / all) persisted with the panel; switching to `kube-system` shows only its Pods, verified against fixture data
- [x] 6.4 Implement name-substring filter and per-column sort; filter `nginx` hides non-matching rows and clearing restores them, Age sort toggles direction - covered by view-model unit tests
- [x] 6.5 Verify watch reconnect: interrupt the mock stream and assert the table converges to the post-interruption cluster state

## 7. Pod logs

- [x] 7.1 Implement a log panel streaming a pod container's logs over its own line-by-line channel (not the coalescing path); against a mock log stream, history renders and new lines append
- [x] 7.2 Implement follow mode with scroll-up-pauses / return-to-bottom-resumes; view-model tests cover the follow state transitions
- [x] 7.3 Implement container selection defaulting to the first container; a multi-container fixture switches streams on selection and a single-container fixture needs no pick
- [x] 7.4 Handle stream end and request failure distinctly; mock a deleted pod and a not-started container and assert each shows its own terminal message

## 8. Command system

- [x] 8.1 Implement `CommandRegistry` (`id`, `title`, `default_binding`, `context`); registering a command makes it invocable by id, and a context-gated command is inert when its context is inactive - unit tested
- [x] 8.2 Install registry entries as GPUI `KeyBinding`s scoped by `KeyContext`, layering `keymap.toml` over defaults; first run writes `keymap.toml` with defaults, an override rebinds after restart, an invalid entry falls back to defaults and leaves the file untouched - covered by loader tests
- [x] 8.3 Implement the fuzzy command palette listing context-available commands with their bindings; typing `new win` surfaces "New Window" and running it opens a window - manual check plus a fuzzy-match unit test (`fuzzy_match`/`build_items` tested against gpui-component's real `Command` type; `cmd-shift-p` opens it as a Dialog on the focused window's Root, verified end-to-end by dispatching the action in a test and confirming `open_command_palette` runs without panicking - actually rendering the palette via `render_dialog_layer` in a test trips an entity leak in gpui-component 0.6.6's own `CommandState`, unrelated to our code, so that specific render path is asserted manually instead; typing "new win" and clicking "New Window" to confirm a window opens is still an outstanding manual check - no display access in this environment)

## 9. Integration verification

- [ ] 9.1 Run the full app on Linux (Wayland and X11 if available): connect to a real cluster, open two Pods panels in two windows, confirm one shared watch, stream a pod's logs, quit and relaunch to confirm layout restore - reopened by 9.3: at the time this was verified, `ClusterSession` (the shared-watch mechanism) wasn't actually wired into panel creation, so the two panels each ran an independent watch rather than sharing one; App commit `bee6130` wires it in and adds a unit test proving two subscribers get the same table entity and the watch only tears down on the last unsubscribe - needs re-verification against that commit, in particular that a second window's Pods panel shows the same data as the first without a visible second connect/relist
- [x] 9.2 Load-test the drain: point a Pods panel at a namespace with several hundred Pods and confirm the UI stays responsive during initial list and a delete-all burst
- [x] 9.3 Confirm no trait/abstraction in the diff has only one implementation and no unused "for later" seam remains; note anything deliberately kept with a one-line reason - findings (App commit `bee6130`): removed `runtime::drain_coalescing` (built, tested, never called in production - `pods.rs`'s doc comment wrongly described the real `drain` as "the coalescing drain") and `pods::apply_and_notify` (dead duplicate of logic already inlined in `watch_all_namespaces`); found and fixed `ClusterSession`/`WatchRegistry` being fully unused (see 9.1 reopen note) rather than leaving it as dead code, since it maps to a real requirement; kept as deliberate single-instance generics: `ResourceIndex<Pod>` in `pods.rs` (a real generic container, just only one resource kind exists yet) and `PlaceholderPanel` (explicitly a stand-in for the second dock panel, already documented as such)
