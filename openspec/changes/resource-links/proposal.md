# Proposal

## Why

Detail views name other objects everywhere: a pod names its node, its owning ReplicaSet, its
ServiceAccount, and the ConfigMaps, Secrets and PVCs it mounts. Today those names are dead text.
Namespace, Controlled By, Node and Service Account are styled as links but not clickable (the
`pod-detail-panel` design deferred click behaviour to "a second, smaller pass"). Volume and
environment references aren't styled at all. So getting from a pod to whatever it references means
going back to the Resource panel and finding the object by hand. That breaks the keyboard-first,
panel-per-object workflow the app is built around, and the gap grows with every detail view added.

## What Changes

- Any field in a resource detail view that references another object the app can display is a
  link. Activating it (by click or keystroke) opens that object's panel, in the same cluster
  context as the panel it was followed from, or focuses that panel if it's already open.
- References to kinds the app can't display yet are plain text, not link-styled. Links never
  lead nowhere. A reference becomes a link as soon as its kind gains a viewer, with no change
  where the reference is shown.
- Pod detail is the first user. It links Namespace, each owner in Controlled By (one link per
  owner, not a comma-joined string), Node, Service Account, and the ConfigMaps, Secrets and PVCs
  its volumes name. It also shows two reference fields it doesn't have yet (as links): image pull
  secrets, and the containers' ConfigMap and Secret references in `envFrom` and `valueFrom`.
- Later detail views (Node, workload controllers, ConfigMap, ...) must represent their references
  the same way, so this is one mechanism rather than one feature per panel.

## Capabilities

### New Capabilities
- `resource-links`: how a detail view represents a reference to another object, when that
  reference is followable, and what following it opens (target panel, cluster context, dedup
  with open panels, keyboard reachability, targets that no longer exist).

### Modified Capabilities
- `pod-detail`: the fields that reference other objects become followable links under
  `resource-links`, owners get one link each, and the view gains image-pull-secret and container
  env-source references.

## Impact

- `app/src/k8s/resource/pod_detail.rs`: `PodFieldValue::Link` carries a typed object reference
  instead of a display string; `format_volume` and the container summary expose their references;
  the projection gains image pull secrets and env sources. (This file is already far over the
  500-line limit. The split planned after `1-window-context-bar` merges should land first, or this
  change will grow it further.)
- `app/src/ui/nav.rs`: a target for "one specific object of a kind" next to the pod-specific
  target, and the one predicate that says whether a kind has a viewer.
- `app/src/util/shell.rs`: opening a followed reference goes through the existing
  `open_target_with_view` dedup/focus path, scoped to the source panel's context.
- No new dependencies.
