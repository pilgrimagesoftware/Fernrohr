# Tasks

Sequencing: start after `1-window-context-bar` merges (it edits `pod_detail.rs`, `nav.rs` and
`shell.rs`), and after `pod_detail.rs` has been split under the 500-line limit in its own PR.
Reference projection code goes into its own module, not back into the detail file.

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

- [ ] 2.1 Add `nav::viewer_for(&ObjectRef) -> Option<NavTarget>` in place of
  `has_concrete_panel`, resolving core `Pod` and `Namespace`. Verify: unit tests for Pod,
  Namespace, and an unviewable kind (`apps/ReplicaSet`) returning `None`, and existing
  `has_concrete_panel` callers still pass their tests.
- [ ] 2.2 Add `ui/link.rs`: render an `ObjectRef` as a link when `viewer_for` resolves it,
  otherwise as plain unstyled text. Use it for every reference pod detail renders. Verify: a
  render-level test shows a Namespace reference is interactive and a ReplicaSet owner reference
  is not.

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

## 5. Full verification

- [ ] 5.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings`, and `cargo test` all
  pass.
- [ ] 5.2 Manual smoke test: from a pod's detail, follow the namespace link by click and by `g`
  (keyboard only: `g`, arrows, Enter, and Escape to cancel),
  follow it again and confirm focus moves to the existing panel rather than duplicating it, and
  confirm references with no viewer (owner ReplicaSet, Node, ConfigMaps) show as plain text.
  **Needs user confirmation against a running build.**
