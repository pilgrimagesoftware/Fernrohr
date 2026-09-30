# Tasks

Sequencing: start after `1-window-context-bar` merges (it edits `pod_detail.rs`, `nav.rs` and
`shell.rs`). Reference projection code goes into its own module, not back into the detail file.
Sections 5 and 6 were added when the scope widened to include viewers (see `design.md` Goals).

## 0. Prerequisite

- [x] 0.1 Split `pod_detail.rs` under the 500-line limit in its own PR, with no behaviour change.
  Verify: the same tests pass before and after, and no file under `pod_detail/` exceeds 500 lines.

## 1. Typed references

- [ ] 1.1 Add `ObjectRef` (group, kind, optional namespace, name) and constructors from
  `OwnerReference`, a same-namespace bare name, and a cluster-scoped name. Verify: unit tests
  cover the owner `apiVersion` → group split (core `v1` → empty group, `apps/v1` → `apps`) and
  namespace inheritance vs `None` for cluster-scoped kinds.
- [ ] 1.2 Replace `PodFieldValue::Link(String)` with `References(Vec<ObjectRef>)` for Namespace,
  Controlled By (one per owner), Node and Service Account. Verify: the projection tests assert
  the typed references, including a two-owner pod yielding two entries.
- [ ] 1.3 Carry references on volume rows (ConfigMap, Secret, PVC, and projected sources) beside
  their text. Verify: `volumes_are_named_and_typed` (or its successor) asserts each backed
  volume's reference, and an `emptyDir` volume has none.
- [ ] 1.4 Project image pull secrets as a field of references, and each container's
  `envFrom`/`valueFrom` ConfigMap/Secret references deduped per object in first-seen order.
  Verify: tests cover a container reading several keys from one ConfigMap (one reference) and
  from both a ConfigMap and a Secret (two).

## 2. Linkability and rendering

- [ ] 2.1 Add a per-context discovery registry (`k8s::cluster::discovery_registry`), loaded once
  the context connects, and `nav::viewer_for(&ObjectRef, Option<&[DiscoveredKind]>) ->
  Option<NavTarget>` in place of `has_concrete_panel`, resolving core `Pod`, `Namespace`, and any
  discovered kind to `NavTarget::Object`. Verify: unit tests for Pod, Namespace, a discovered
  `apps/ReplicaSet` resolving to `Object`, an undiscovered kind (and any non-Pod kind before
  discovery loads) returning `None`, and existing `has_concrete_panel` callers still pass their
  tests.
- [ ] 2.2 Add `ui/link.rs`: render an `ObjectRef` as a link when `viewer_for` resolves it,
  otherwise as plain unstyled text. Use it for every reference pod detail renders. Verify: a
  render-level test shows a Namespace reference is interactive and a reference to an
  undiscovered kind is not.

## 3. Following a link

- [ ] 3.1 Give `open_target_with_view` an explicit context parameter (the window's context stays
  the default at existing call sites) and add a `FollowReference { context_name, target }`
  action that `MainWindow` handles through it. Verify: shell tests that following a Pod
  reference opens its detail panel, following it again focuses the same panel, and the source
  panel's context is used, not the window's.
- [ ] 3.2 Wire link clicks in `ui/link.rs` to dispatch `FollowReference` with the source panel's
  `PanelScope.context_name`. Verify: a test clicks a reference in a pod detail panel and asserts
  the target panel is open and focused.
- [ ] 3.3 Following a Namespace reference opens (or focuses) the Pods list scoped to that
  namespace. Verify: a shell test asserts the opened panel's namespace scope.

## 4. Keyboard

- [ ] 4.1 Register `links.go_to` (default `g`, gated to the detail panel's key context, no menu
  slot) and show its key in the panel's hint bar via `Kbd::binding_for_action`, hidden when the
  view has no followable references. Verify: a registry test asserts the command's id, context
  and binding, and a render test asserts the hint appears only with followable references.
- [ ] 4.2 Build the "Go to…" picker in `ui/link.rs`: lists the view's followable references
  (kind, name, source field), filters as you type, arrow keys and clicks move one selection
  (hover never does, gated on `window.last_input_was_keyboard()`), Enter or click follows, and
  Escape closes with focus back on the detail panel. Verify: tests using
  `VisualTestContext::simulate_keystrokes` press `g`, type a filter, `down`, `enter` and assert
  `FollowReference` fired for the chosen reference; press `g` then `escape` and assert focus is
  back on the panel; and simulate a hover and assert the selection didn't move.

## 5. Generic object viewer

- [ ] 5.1 Move the row layout and the text/chips/badges/list/reference value renderers pod detail
  uses into a shared `ui/detail` module, with pod detail calling it. Verify: pod detail's render
  tests pass unchanged.
- [ ] 5.2 Add `NavTarget::Object` (discovered kind, optional namespace, name) and the
  `ObjectDetailPanel` fetch: one `DynamicObject` `get` through the kind's `ApiResource`, plus its
  events, into Loading/Loaded/NotFound/Failed. Verify: a fixture-API-server test covers a found
  namespaced object, a found cluster-scoped object, forbidden events, and a 404.
- [ ] 5.3 Render the panel: Overview (created, name, namespace link, labels, annotations, one
  link per owner), the kind's sections, Events, and a `y` YAML toggle with hint-bar keys and `g`
  go-to, both registry commands gated to the panel's key context. Verify: render tests for the
  Overview rows and owner links, and `simulate_keystrokes` tests for `y` and `g`.
- [ ] 5.4 Open, dedup and restore: `nav::add_panel` builds it for `NavTarget::Object`,
  `open_target_with_view` dedups it per context, and the dock restores it (`ObjectDetail`).
  Verify: shell tests that following a ReplicaSet owner opens one panel and following it again
  focuses it, and a dump/restore round-trip test.

## 6. Kind-specific sections

- [ ] 6.1 Node, ConfigMap and PersistentVolumeClaim sections (PVC's volume and storage class as
  references). Verify: projection tests per kind from fixture objects.
- [ ] 6.2 Secret section and redaction: type and key sizes in the structured view, and `data`,
  `stringData` and the last-applied-configuration annotation replaced with size placeholders in
  the stored object, so the YAML view can't show a value. Verify: tests assert no fixture value
  appears in either view's output, and the sizes are right.
- [ ] 6.3 ServiceAccount (secrets and image pull secrets as references) and workload sections
  (ReplicaSet, Deployment, StatefulSet, DaemonSet, Job: replicas, selector, conditions). Verify:
  projection tests per kind, including a ReplicaSet whose Deployment owner is a reference.

## 7. Full verification

- [ ] 7.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` all
  pass.
- [ ] 7.2 Manual smoke test: from a pod's detail, follow the namespace link by click and by `g`
  (keyboard only: `g`, arrows, Enter, and Escape to cancel), follow it again and confirm focus
  moves to the existing panel rather than duplicating it; follow the owner ReplicaSet, then its
  Deployment; follow the node, a mounted ConfigMap, and a Secret (confirming no value shows in
  fields or YAML); and follow a reference to a Secret that doesn't exist.
  **Needs user confirmation against a running build.**
