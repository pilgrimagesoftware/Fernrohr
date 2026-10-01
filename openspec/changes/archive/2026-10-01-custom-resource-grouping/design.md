# Design

## Context

`k8s/cluster/discovery.rs`'s `discover_kinds` already returns `DiscoveredKind`s sorted by group
then kind with core first, and the Resource panel (`ui/panel/resource/`) buckets them into seven
fixed categories with a per-window `HashSet<Category>` of collapsed sections (`resource.rs`). The
accepted `resource-browser` spec forbids deriving a *section* from the API group alone, so API-group
grouping is applied one level down, inside Custom Resources, where every kind is by definition
outside the built-in categories.

## Goals / Non-Goals

**Goals:**
- Sub-group Custom Resources by API group with zero change to discovery/sort data; grouping is a
  rendering and interaction concern.
- Keep subgroup collapse state cheap and consistent with existing section state.

**Non-Goals:**
- No change to the seven categories or their order.
- No persistence of collapse state to the preference file or across windows.
- No user-defined groupings; subgrouping is strictly by API group as discovery reports it.

## Decisions

- **Subgroup collapse state is a per-window `HashSet<String>` (API group name) next to the
  existing `HashSet<Category>`.** Alternative considered: per-cluster-connection session state, as
  first proposed - rejected for consistency with section collapse, which the `resource-browser`
  spec already fixes as per-window and not persisted.
- **Subgroups default to collapsed, so the stored set tracks *expanded* subgroups** (starting
  empty) rather than collapsed ones; groups discovered later are collapsed with no extra work.
  Collapse/expand all clears or fills that set.
- **Filtering computes visibility per-kind first, then derives subgroup visibility and forced
  expansion from the matched set**, without mutating the stored collapsed set, so clearing the
  filter restores prior state for free.
- **Subgroup headers join the panel's existing flattened focus order** as one item each;
  collapsing removes that subgroup's kinds from the order but keeps its header.

## Risks / Trade-offs

- [A cluster with very many CRD groups yields many subgroup headers] → Out of scope; collapsing
  Custom Resources itself still hides all of them.
