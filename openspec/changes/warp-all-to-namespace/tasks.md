# Tasks

## 1. Context default state

- [ ] 1.1 Add an optional per-context default namespace field to persisted workspace/session state and verify existing state files still deserialize.
- [ ] 1.2 Apply the context default when constructing a new namespaced panel and verify a panel opened after setting the default starts in that namespace.

## 2. Commands and propagation

- [ ] 2.1 Register the "Warp all to namespace" action with the existing command system and verify it appears in the palette for a Pods panel.
- [ ] 2.2 Update all open namespaced panels in the active context and set the context default, leaving cluster-scoped panels unchanged; verify with focused-panel and multi-panel tests.

## 3. Verification

- [ ] 3.1 Run the relevant Rust unit and integration tests and verify existing single-panel warp behavior is unchanged.
