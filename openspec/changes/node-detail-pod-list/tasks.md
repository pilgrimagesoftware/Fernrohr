# Tasks

## 1. Fetch pods on the node

- [ ] 1.1 Add a `list_pods_on_node(client, node_name) -> Result<Vec<ObjectRef>, String>` helper
  (near `events::list` or in `fetch.rs`) that lists `Pod`s cluster-wide with field selector
  `spec.nodeName=<node_name>` and maps each to an `ObjectRef`; verify with a unit test against a
  fake/mocked client (or the project's existing kube-mocking fixtures) covering a non-empty
  result, an empty result, and a listing error mapped to a readable message.
- [ ] 1.2 Add `node_pods: Option<Result<Vec<ObjectRef>, String>>` to `ObjectDetailState::Loaded`
  and `ObjectFetch::Found`, populated by `fetch_object` only when `target.kind` is Node (`None`
  otherwise); verify with a test that `fetch_object` for a non-Node target returns `None` and for
  a Node target returns `Some(_)`.

## 2. Node section gains the Pods field

- [ ] 2.1 Change `cluster::node`'s signature to accept the fetched pod result and append a `Pods`
  `ObjectField::references(...)` on success (empty vec renders as an empty field, not omitted) or
  a text field stating the error on failure; verify with the existing `sections::tests` pattern
  for Node, adding cases for non-empty, empty, and error.
- [ ] 2.2 Update `sections_for`'s Node arm and its one call site in `panel.rs` to pass the new
  `node_pods` value through; verify `cargo build` succeeds and existing `sections_for` tests for
  every other kind are unaffected (no signature change needed for their arms).

## 3. Integration and documentation

- [ ] 3.1 Add or update an integration test in `object_detail/tests/panel.rs` opening a Node's
  panel with fixture pods on that node, asserting the Pods field lists them as links and that
  following one opens that pod's detail panel (reusing the existing resource-links-follow test
  helper).
- [ ] 3.2 Update `cluster.rs`'s module doc comment ("A Node: where it is reachable, what it
  offers, how it's doing, what it runs...") if its current wording doesn't already cover "what it
  runs" accurately once the Pods field lands; verify by re-reading the comment against the final
  `node()` implementation.
- [ ] 3.3 Run `cargo fmt && cargo clippy -- -D warnings && cargo test` and fix any failures.
