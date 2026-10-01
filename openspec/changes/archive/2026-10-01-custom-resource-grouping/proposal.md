# Proposal

## Why

The Resource panel already groups discovered kinds into seven fixed category sections (Workloads
… Access Control, Custom Resources), but Custom Resources is itself one flat list of every CRD the
cluster serves. As clusters accumulate CRDs (cert-manager, Istio, Argo, vendor operators), that
section grows long and mixes unrelated APIs, making a kind hard to find by scanning. Sub-grouping
it by API group turns an alphabetic scan into a recognizable landmark search.

## What Changes

- Within the Custom Resources section only, present kinds as subgroups headed by API group,
  ordered alphabetically, with kinds in the existing sort order. Any core-group kind that falls
  into Custom Resources (uncategorized) is shown first. The seven top-level categories are
  unchanged.
- Collapse/expand each subgroup, held per window alongside the existing section collapse state.
- The panel's existing filter works across subgroups: while a filter is active only subgroups
  with a match are shown, a collapsed subgroup with a match is shown expanded, and its prior
  state returns when the filter clears.
- Keyboard navigation includes subgroup headers in the existing focus order, and a keybinding
  toggles the focused subgroup's collapsed state.

## Capabilities

### New Capabilities

- `resource-group-navigation`: API-group subgrouping within the Resource panel's Custom
  Resources section, including subgroup collapse state and how filtering and keyboard navigation
  interact with subgroups.

### Modified Capabilities

(none - `resource-browser`'s category sections and "a section SHALL NOT be inferred from the API
group alone" requirement are preserved; subgroups live inside a section)

## Impact

- `App/src/ui/panel/resource/{category,section}.rs` and `resource.rs` (rendering, collapse state).
- `App/src/k8s/cluster/discovery.rs`'s existing group-then-kind sort (reused, not changed).
