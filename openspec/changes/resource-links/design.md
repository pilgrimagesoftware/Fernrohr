# Design

## Context

Current state (see proposal.md for why this matters):

- `NavTarget` has three variants: `Kind(DiscoveredKind)` (a list panel), `Logs`, and
  `Pod(PodRef)` (one pod's detail). Only core `v1 Pod` has a concrete panel
  (`nav::has_concrete_panel`). Every other discovered kind opens `PlaceholderPanel`.
- `MainWindow::open_target_with_view` (`util/shell.rs`) is where opening happens, and it already
  dedups: it builds a `PanelKey` from the `PanelScope` (target + `context_name` + namespaces) and
  focuses a matching open panel instead of adding another. It always scopes to the *window's*
  `context_name`.
- Pod detail's `PodFieldValue::Link(String)` is only colored text. Owners are comma-joined into
  one string. Volumes are formatted strings in a `List`. Env sources and image pull secrets aren't
  projected at all.
- Pod detail opening is driven by a dataless `ShowPodDetail` action that reads the `SelectedPod`
  global. That works for "the selected row" but not for "this particular reference".

## Goals / Non-Goals

**Goals:**
- One typed reference representation, one "does this kind have a viewer" decision, and one render
  helper, shared by every detail view present and future.
- Following a reference reuses the existing open/dedup/focus path, not a second one.
- **Viewers for what pod detail references** (scope widened at the user's request, 2026-09-30,
  replacing the original "no new viewers" non-goal): a generic single-object viewer for any kind
  the cluster's discovery reports, so every reference to a discovered kind is live on the day this
  lands, with structured sections for the kinds pod detail names most: Node, ConfigMap, Secret,
  PersistentVolumeClaim, ServiceAccount, and the workload owners (ReplicaSet, Deployment,
  StatefulSet, DaemonSet, Job). Specified by the new `object-detail` capability.

**Non-Goals:**
- **List panels for other kinds.** Selecting ConfigMaps (etc.) in the Resource panel still opens
  the placeholder. Links need single-object viewers, not lists; a generic list panel is the next
  change.
- **Live updates in the object viewer.** It reads its object once, like pod detail does, and
  shows the not-found state for a 404. A watch per open detail panel is a separate decision.
- Opening in a new window or split (modifier-click). Opening follows the normal open path.
- Links out of the YAML view. Only the structured view has fields to link.
- Showing secret values, anywhere. See "Secrets never show values" below.

## Decisions

### A typed `ObjectRef`, built at projection time

`ObjectRef { group: String, kind: String, namespace: Option<String>, name: String }`, built in
the projection (`pod_fields` and its helpers) from the API objects' own reference shapes:
`OwnerReference` (`apiVersion` → group), `spec.nodeName` (core `Node`, cluster-scoped),
`spec.serviceAccountName`, volume sources, `envFrom`/`valueFrom`, `imagePullSecrets`. It's parsed
once at the boundary, so render code never re-derives a kind from a formatted string.

- **No version.** Identity for "which viewer" is `(group, kind)`. Owner references carry a
  version, but a viewer is per kind, and matching on version would make a
  `apps/v1` vs `apps/v1beta2` difference stop a link from resolving.
- **Namespace filled in at projection.** A namespaced object referencing another by bare name
  (every pod reference except Node) gets the pod's namespace. Cluster-scoped kinds (Node) get
  `None`. The render side never reasons about scope.
- *Alternative considered:* keep `Link(String)` and parse `Kind/name` back out when clicked.
  Rejected. It's the "validate, don't parse" shape that loses group and namespace, and the
  owner comma-join already shows how such strings drift.

`PodFieldValue::Link(String)` becomes `PodFieldValue::References { targets, qualified }` (one
entry for Node/Namespace/Service Account, several for owners). `qualified` says whether each
reads as `Kind/name`: a row whose label already names the kind reads better without it. Volume rows and container cards carry
`Vec<ObjectRef>` beside their text instead of folding the name into it.

### One predicate decides linkability: `viewer_for`

`fn viewer_for(&ObjectRef) -> Option<Destination>` in `ui/viewer.rs` is the single place that knows
which kinds a reference can be followed to. A `Destination` is a `NavTarget` plus the namespace
scope to open it with:

- core `Pod` → `NavTarget::Pod`, which keeps its own richer panel;
- core `Namespace` → the Pods list scoped to that namespace;
- once section 5 lands, any other `(group, kind)` present in the context's discovery →
  `NavTarget::Object`, the generic viewer, carrying the discovered kind (so the panel knows the
  version, plural and scope it needs to build a `kube` `ApiResource`) plus namespace and name;
- anything else, including every non-Pod kind while discovery hasn't loaded yet → `None`, plain
  text.

A new *kind-specific* viewer is registered where the generic viewer picks its sections
(`object_detail::sections_for`), not at any reference site, which is how the spec's "a kind gains
a viewer" scenario holds.

- `has_concrete_panel` stays. It answers a different question (which *list* panel a kind gets),
  and the Resource panel, which `resource-panel-grouping` is rewriting, calls it.
- `Pod` is not folded into `NavTarget::Object`. It already has its own panel, dock-restore name
  and tests; folding it in would be churn with no user-visible change. `viewer_for` is the one
  place both are reached from.
- *Alternative considered:* a static allow-list of viewable kinds. Rejected: with a generic viewer,
  "the cluster reports this kind" is the real test, and it's the one that makes a removed CRD fall
  back to plain text.

### A per-context discovery registry

Discovery results live today only inside each window's `ResourcePanel`. Links need them in any
panel, so `k8s::cluster::discovery_registry` holds one `Entity<DiscoveredKinds>` per context, built
the same way `NamespaceRegistry` holds namespace lists: created on first use, loaded once the
context connects, observed by whoever renders links so a reference turns into a link when
discovery lands. The `ResourcePanel` keeps its own copy for now; `resource-panel-grouping` is
rewriting that file, and moving it onto the registry is a small follow-up once that lands.

### Opening goes through `open_target_with_view`, scoped to the source panel

A shared `ui/link.rs` renders an `ObjectRef` as a link (if `viewer_for` says so) or plain text.
Activating a link dispatches a data-carrying action
`FollowReference { context_name, target: ObjectRef }` that `MainWindow` handles
(`util/shell/follow.rs`) by calling `open_target_in` with that `context_name`, not the window's.
That makes the dedup/focus behaviour in the spec the existing behaviour rather than new logic.
`open_target_with_view` becomes a wrapper over `open_target_in(target, view, context, namespaces)`,
passing the window's own choice of context, as before, at its existing call sites. A context the
window doesn't hold is refused, not substituted. This matters now: `1-window-context-bar` introduces
multi-context windows, where the window's context and a panel's context can differ.

- *Alternative considered:* set a global "selected reference" and dispatch a dataless action, as
  `ShowPodDetail` does with `SelectedPod`. Rejected: a link is its own selection, and a shared
  global would race between two panels.

### Keyboard: a "Go to…" picker per detail view

Per the keyboard-first rule (`.claude/rules/keyboard-first.md`), both routes are specified: click
a link, or press `g` for a "Go to…" picker. The picker is a registry `Command`
(`links.go_to`, default `g`, gated to each detail panel's key context, no menu slot), so it gets a
palette entry, a `keymap.toml` override, and a hint-bar key read from the live keymap with
`Kbd::binding_for_action`. It lists the view's followable references (kind, name, the field
they came from), filters as you type, moves one selection with arrow keys or clicks, follows on
Enter or click, and closes on Escape with focus back on the detail panel.

- **Hover never moves the selection.** gpui-component's `Command` list selects on hover, so the
  picker follows the cluster picker's `follow_keyboard` pattern: it only takes on a highlight
  change when `window.last_input_was_keyboard()` is true.
- It's built once in `ui/link.rs` from the same `Vec<ObjectRef>` the view renders, so the picker
  and the visible links can't disagree.
- The picker opens in the window root's dialog layer, which sits outside the workspace's element
  tree, so following refocuses the detail view first and dispatches `FollowReference` from there.
- Enter follows the picker's own selection when the input was the keyboard; a click follows the
  clicked row. `Command` reports its hover-following highlight for both.
- `FollowReference` stays an internal, data-carrying action and not a registry command: it has no
  meaning without a specific reference, the same reason `ShowPodDetail` isn't registered. The
  user-facing command is `links.go_to`.
- *Alternative considered:* Tab/arrow focus cycling through links in place. Rejected for now:
  focus is already hard to see (see `per-tab-close-button`), a pod with many env sources would make
  cycling slow, and the panels use `tab_stop(false)` on in-body controls.

### Env references deduped per container

`envFrom` (`configMapRef`/`secretRef`) and `valueFrom` (`configMapKeyRef`/`secretKeyRef`) collapse
to one reference per object per container, in first-seen order, so twenty keys from one ConfigMap
read as one link. Projected-volume sources (`projected.sources[].configMap/secret`) are volume
references like any other.

### The generic object viewer: `ObjectDetailPanel`

A new dock panel over `NavTarget::Object`, fetched with one `Api::<DynamicObject>` `get` built from
the discovered kind's `ApiResource` (namespaced or cluster-wide by the kind's scope), plus the
events naming it (`involvedObject.kind/name/namespace/uid`, the same selector shape pod detail
uses). States mirror `PodDetailState`: Loading, Loaded, NotFound, Failed.

- **Structured view:** Overview (created, name, namespace as a link, labels, annotations, owners
  as one link each) followed by the kind's own sections, then Events. **YAML view:** the manifest,
  toggled with `y` like pod detail. Hint bar, `g` go-to, and focus behave as in pod detail.
- **Shared row pieces.** The object viewer and pod detail both draw labelled rows of text, chips,
  badges, lists and references. The row layout and those value renderers move to a shared
  `ui/detail` module both panels call, instead of a second copy in the new panel. Each panel keeps
  its own field model: pod detail's tabs and container/managed-field cards are its own.
- **Dock restore:** `panel_name` "ObjectDetail", dumping context, group/version/kind/plural/scope,
  namespace and name, the same way `PlaceholderPanel` dumps a kind.

### Kind-specific sections, projected from typed objects

`sections_for(&DiscoveredKind, &DynamicObject)` deserializes the object into its `k8s-openapi`
type for the kinds listed under Goals and projects a few sections each (Node: addresses,
capacity/allocatable, conditions, node info, taints; ConfigMap: data keys and values; Secret: type
and keys; PVC: status, capacity, access modes, storage class and volume as references;
ServiceAccount: secrets and image pull secrets as references; workloads: replicas, selector,
conditions). A kind not listed, or an object that doesn't deserialize, gets metadata only.
Projection stays pure (`object in, fields out`) and testable without a window, like `pod_fields`.
The Secret section is the exception to "typed": it reads the redacted object's JSON, because the
size placeholders aren't the base64 `k8s-openapi`'s `Secret` deserializes. Condition tone is per
condition, not per status: `Ready` is good news when True, while a Node's pressure conditions and
a workload's `ReplicaFailure`/`Failed` are good news when False.

### Secrets never show values

A Secret viewer shows each key's name and decoded byte length, never the value, in the structured
view **and** the YAML view: before rendering, `data` and `stringData` values are replaced with a
`<redacted: N bytes>` placeholder, and so is the `kubectl.kubernetes.io/last-applied-configuration`
annotation (which holds the full manifest, values included, when `kubectl apply` created the
Secret). Redaction happens on the fetched object before it's stored in the panel, so no render
path can reach a value.

## Risks / Trade-offs

- [A reference is plain text until the context's discovery loads] → The link registry is
  observed, so the reference becomes a link as soon as discovery lands; Pod and Namespace
  references resolve without discovery.
- [Every object viewer needs `get` on its kind] → A forbidden `get` is the Failed state with the
  API's own message, same as pod detail; nothing is fetched until the link is followed.
- [`pod_detail.rs` is ~2,800 lines against a 500-line limit, and this change adds to it] → Split it
  first, in its own PR (section 0) after `1-window-context-bar` merges. The reference projection
  goes into its own module either way.
- [`open_target_with_view` signature change conflicts with `1-window-context-bar`] → Sequence
  after it merges. The change is additive (a context argument with the window's as default).
- [A reference to a kind whose group isn't in discovery (CRD removed)] → `viewer_for` returns
  `None`, so it shows as plain text, same as any unviewable kind.
