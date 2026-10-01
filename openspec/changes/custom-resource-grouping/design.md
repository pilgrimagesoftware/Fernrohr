# Design

## Context

`cluster/discovery.rs`'s `discover_kinds` already returns `DiscoveredKind`s sorted by group then
kind with core first (`kinds_sort_by_group_then_kind_with_core_first`), so the ordering this
change needs already exists; only the picker's rendering and interaction model change. The picker
(`ui/picker.rs`) is a per-cluster-connection view, so "remember collapsed state for the session"
maps naturally onto state already scoped to that connection.

## Goals / Non-Goals

**Goals:**
- Turn the existing sorted flat list into a sectioned list with zero change to the underlying
  discovery/sort data; grouping is purely a rendering and interaction concern.
- Keep collapse state cheap and session-scoped (no new config file, no cross-session
  persistence).

**Non-Goals:**
- No change to how a kind is picked once found (opening a panel for it is unchanged).
- No persistence of collapsed state across app restarts or across cluster connections.
- No user-defined custom groupings; grouping is strictly by Kubernetes API group as discovery
  reports it.

## Decisions

- **Collapsed state lives on the per-cluster-connection session, as a `HashSet<ApiGroup>`.**
  Alternative considered: storing it on the picker view itself — rejected because the picker view
  is recreated each time it opens, which would reset collapse state every time rather than only
  at the next cluster connection, as the spec requires.
- **Filtering computes visibility per-kind first, then derives group visibility and forced
  expansion from the matched set**, rather than filtering within an already-rendered tree. This
  keeps "a group with no match is hidden entirely" and "a matching collapsed group shows
  expanded" simple set operations over the same list `discover_kinds` already produced.
- **Keyboard navigation flattens the grouped-and-filtered view into one ordered list of focusable
  items (headers and kinds)** before applying arrow-key movement, reusing the picker's existing
  single-list focus/arrow-key handling instead of writing a 2D (group, index) navigation model.
  A group header counts as one item in that list; collapsing it removes its kinds from the list
  but keeps the header.

## Risks / Trade-offs

- [A cluster with a very large number of CRD groups could still produce a long list of headers]
  → Out of scope for this change; the spec only requires grouping, not search/favorites. Can be
  revisited if real clusters show this in practice.
- [Reusing a flattened single-list focus model means "jump to next group" isn't a distinct
  keyboard affordance] → Acceptable: arrow-key navigation already crosses group boundaries
  (per spec), and the toggle-group keybinding covers the collapse/expand case explicitly asked
  for; a dedicated "next group" jump can be added later without a spec change if requested.
