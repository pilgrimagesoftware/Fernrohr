## 1. Workspace and dependency setup

- [ ] 1.1 Create the Cargo workspace and an app crate; `cargo build` succeeds with an empty GPUI window that opens on macOS
- [ ] 1.2 Add and pin GPUI + gpui-component to specific git revisions; a gpui-component `Dock` renders in the window and `cargo build` is reproducible from a clean checkout
- [ ] 1.3 Add `tokio` (multi-thread), `kube`, `kube-runtime`, `k8s-openapi`, `dirs`, `serde`, `toml`, and an i18n crate; `cargo build` succeeds and `cargo tree` shows no duplicate major versions of `tokio` or `hyper`
- [ ] 1.4 Add a Linux CI job (build + `cargo test`) alongside macOS; both legs pass on the empty-window commit

## 2. Platform paths and config loading

- [ ] 2.1 Implement `paths.rs` resolving state, preferences, and cache roots via `dirs`, using a reverse-domain id on macOS and `fernrohr` elsewhere; unit tests assert each resolved path ends with the expected dir name and falls back to `.` when the platform dir is unavailable
- [ ] 2.2 Implement a generic typed-TOML `load()`/`save()` helper: first call with no file writes defaults, a corrupt file is left untouched and defaults are returned; unit tests cover create-default, round-trip, and parse-failure-keeps-file
- [ ] 2.3 Define `ui.toml`, `window_state.toml`, and `workspace.toml` structs on the helper; `cargo test` covers serde round-trips and unknown-field tolerance

## 3. The runtime spine

- [ ] 3.1 Build the `tokio` runtime at startup, store its handle in a GPUI global, and expose a helper to spawn a `tokio` task that streams items back over a bounded channel; an integration test spawns a task that emits N items and asserts the foreground drain receives them in order
- [ ] 3.2 Implement the coalescing drain: fold consecutive updates to the same key before applying; a test feeds an interleaved add/update/delete burst and asserts the resulting index matches the final state with bounded intermediate work
- [ ] 3.3 Implement the per-`(cluster, kind)` entity index (`Vec` + `HashMap<Uid,usize>`) with apply-`Applied`/apply-`Deleted`; unit tests cover insert, in-place update, delete, and missing-on-delete

## 4. Cluster connection

- [ ] 4.1 Load kubeconfig contexts via `kube`'s loader honoring `$KUBECONFIG` then `~/.kube/config`; a test with a fixture kubeconfig lists all context names and asserts the file's bytes and mtime are unchanged after load
- [ ] 4.2 Connect a selected context on the `tokio` runtime, building the client and running one probe request; connection state (connecting / connected / failed-with-reason) is observable in the UI, verified against a mock API server for success, unreachable, and exec-plugin-failure cases
- [ ] 4.3 Run API discovery on connect and expose the discovered kinds; a test against recorded discovery fixtures asserts `Pod` is present and the call runs off the foreground executor

## 5. App shell

- [ ] 5.1 Implement the docked panel workspace: add, split, resize, and close panels; manual check - a second panel splits the area and the separator drags
- [ ] 5.2 Implement multiple windows via a "New Window" command; opening one yields an independent workspace and closing the last window persists state then exits - verified manually and by a state-file assertion on exit
- [ ] 5.3 Persist and restore per-window geometry and panel descriptors through `workspace.toml`; quit-and-relaunch restores two windows with the same panels, and a corrupt `workspace.toml` yields one default window with the file untouched
- [ ] 5.4 Skip unknown panel `kind`s on restore with a log line; a hand-edited `workspace.toml` with a bogus kind still restores the known panels

## 6. Resource browser (Pods)

- [ ] 6.1 Implement the `WatchRegistry` on `ClusterSession` with `subscribe`/`unsubscribe` refcounting per kind; tests assert one stream for two subscribers and teardown on the last `unsubscribe`
- [ ] 6.2 Implement the Pods panel table (name, namespace, ready, status, restarts, age) bound to the kind entity; against a mock stream, initial rows populate and a later `Applied`/`Deleted` adds/removes a row within the drain cycle
- [ ] 6.3 Implement namespace scoping (single / all) persisted with the panel; switching to `kube-system` shows only its Pods, verified against fixture data
- [ ] 6.4 Implement name-substring filter and per-column sort; filter `nginx` hides non-matching rows and clearing restores them, Age sort toggles direction - covered by view-model unit tests
- [ ] 6.5 Verify watch reconnect: interrupt the mock stream and assert the table converges to the post-interruption cluster state

## 7. Pod logs

- [ ] 7.1 Implement a log panel streaming a pod container's logs over its own line-by-line channel (not the coalescing path); against a mock log stream, history renders and new lines append
- [ ] 7.2 Implement follow mode with scroll-up-pauses / return-to-bottom-resumes; view-model tests cover the follow state transitions
- [ ] 7.3 Implement container selection defaulting to the first container; a multi-container fixture switches streams on selection and a single-container fixture needs no pick
- [ ] 7.4 Handle stream end and request failure distinctly; mock a deleted pod and a not-started container and assert each shows its own terminal message

## 8. Command system

- [ ] 8.1 Implement `CommandRegistry` (`id`, `title`, `default_binding`, `context`); registering a command makes it invocable by id, and a context-gated command is inert when its context is inactive - unit tested
- [ ] 8.2 Install registry entries as GPUI `KeyBinding`s scoped by `KeyContext`, layering `keymap.toml` over defaults; first run writes `keymap.toml` with defaults, an override rebinds after restart, an invalid entry falls back to defaults and leaves the file untouched - covered by loader tests
- [ ] 8.3 Implement the fuzzy command palette listing context-available commands with their bindings; typing `new win` surfaces "New Window" and running it opens a window - manual check plus a fuzzy-match unit test

## 9. Integration verification

- [ ] 9.1 Run the full app on Linux (Wayland and X11 if available): connect to a real cluster, open two Pods panels in two windows, confirm one shared watch, stream a pod's logs, quit and relaunch to confirm layout restore
- [ ] 9.2 Load-test the drain: point a Pods panel at a namespace with several hundred Pods and confirm the UI stays responsive during initial list and a delete-all burst
- [ ] 9.3 Confirm no trait/abstraction in the diff has only one implementation and no unused "for later" seam remains; note anything deliberately kept with a one-line reason
