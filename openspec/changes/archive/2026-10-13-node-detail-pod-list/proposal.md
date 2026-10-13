# Proposal

## Why

A Node's detail panel today shows its addresses, capacity, conditions, node info, and taints, but
not what's actually running on it. Figuring out which pods a node carries - the thing you usually
want to know right before or during a node drain, cordon, or capacity investigation - currently
means leaving the Node panel and filtering the cluster-wide Pods list by hand.

## What Changes

- The Node section of the object detail panel gains a "Pods" field listing every pod whose
  `spec.nodeName` matches the node, as `resource-links` references (so each opens or focuses that
  pod's detail panel, following the existing link convention).
- The object detail panel's fetch path, when the target is a Node, additionally lists pods
  cluster-wide filtered by field selector `spec.nodeName=<node>`, alongside its existing
  object-plus-events fetch, and reports a failure to list them the same way it already does for
  events (readable message, not a panel-wide error).
- An empty result (no pods scheduled - a freshly joined or cordoned-and-drained node) shows the
  Pods field as empty, not omitted, so "no pods" is distinguishable from "didn't load".

## Capabilities

### New Capabilities

(none)

### Modified Capabilities

- `object-detail`: the Node kind-specific section gains a Pods field; the panel's data-fetch
  behavior for a Node target gains a second list query and its own failure handling, mirroring
  the existing events fetch.

## Impact

- `App/src/k8s/resource/object_detail/fetch.rs`: `fetch_object` gains a pods-on-node list call for
  Node targets, reusing the `Result<Vec<_>, String>`-alongside-the-object pattern already used for
  events.
- `App/src/k8s/resource/object_detail/sections/cluster.rs`: `node()` gains a `Pods` field built
  from the fetched pod list, via the existing `FieldValue::References` variant.
- `App/src/k8s/resource/object_detail/sections/mod.rs` and `panel.rs`: `sections_for` needs the
  fetched pod list threaded alongside the object for the Node case (every other kind's section
  builder is unaffected).
- `App/src/k8s/resource/object_detail/model.rs`: no change - `FieldValue::References` already
  supports this.
