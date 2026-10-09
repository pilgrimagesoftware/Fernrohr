# Tasks

## 1. Load phase

- [ ] 1.1 Add a shared `LoadPhase` tracker (`Loading { received }` and `Loaded`, refresh detection on a later `Init`) and use it in the object list, Pods, and events browser stores; verify unit tests feeding `Init`, `InitApply`, and `InitDone` sequences, including a relist after `Loaded`.

## 2. Rendering

- [ ] 2.1 Render the first-load indicator (spinner, "Loading <Kind>…", received count) after a 300 ms delay, and the empty and filtered-empty states, in the three panels; verify render tests for loading before and after the delay, a received count, an empty namespaced kind, an empty cluster-scoped kind, and a filter matching nothing.
- [ ] 2.2 Render the header refreshing indicator during a relist with rows on screen, keeping the rows; verify a test that changes scope on a loaded panel and asserts the rows stay until `InitDone` replaces them.

## 3. Integration

- [ ] 3.1 Verify `cargo fmt -- --check`, `cargo clippy --all-targets -- -D warnings`, `CARGO_BUILD_WARNINGS=deny cargo build --workspace --locked`, and `cargo test` pass.
