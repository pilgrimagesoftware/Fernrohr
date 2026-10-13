# Tasks

> Shipped in Fernrohr-App PR 202 (closes Fernrohr-App#186) with a different design from the
> one first planned here. Instead of a one-shot `spec.nodeName` field-selector fetch rendered
> as a `Pods` references field, the Node panel embeds the Pods panel itself, scoped to the node,
> in a region below its sections. That reuses the context's shared all-namespaces pod watch, so
> the list stays live and costs no extra API calls. The tasks below record what shipped.

## 1. Pods on the node

- [x] 1.1 Scope the Pods panel to one node (`PodsPanel::scoped_to_node`), filtering the
  context's shared all-namespaces pod watch (`ClusterRegistry::subscribe_pods`) client-side by
  `spec.nodeName` (`pods::rows::matches_node`), with no extra watch or list call; covered by
  `pods/rows/tests.rs`.
- [x] 1.2 Embedded, the panel leaves out its own context header, namespace picker and Tab trap,
  and says "No pods are scheduled on this node." when the node runs none; its Connecting and
  Failed states come from the Pods panel's existing handling.

## 2. Node detail region

- [x] 2.1 `object_detail::node_pods` builds the region once for the core, cluster-scoped `Node`
  kind only (`None` for every other kind) and renders it below the Node's sections in the
  structured view.
- [x] 2.2 The region keeps the Pods panel's row actions (`d` detail, `l` logs, quick look) and
  `/` search, dispatched in the Node's own cluster context, and adds
  `object_detail.node_pods.open_detail`, bound to `enter` in `NodePodsTable && !Input`, to open
  the selected pod's detail.

## 3. Integration and documentation

- [x] 3.1 Integration tests in `object_detail/tests/node_pods.rs` and
  `util/shell/node_pods_window_tests.rs`, with `FakeCluster` answering a cluster-scoped Node
  GET, cover the region listing the node's pods and opening one.
- [x] 3.2 `node_pods.rs`'s module doc comment describes the region, why it embeds the Pods panel
  rather than duplicating it, and the one `enter` gap it closes.
- [x] 3.3 Run `cargo fmt && cargo clippy -- -D warnings && cargo test` and fix any failures.
