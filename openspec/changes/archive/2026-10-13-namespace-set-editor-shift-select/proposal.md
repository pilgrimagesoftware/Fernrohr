# Proposal

## Why

The namespace set editor (`namespace-sets`) lists the cluster's namespaces with one click per row
to add or remove it from the set. For a set spanning a contiguous run of namespaces - everything
between `team-a` and `team-f`, say - that means one click per namespace. Shift-click range
selection, the standard convention for multi-select lists, cuts that to two clicks.

## What Changes

- Shift-clicking a row in the set editor's namespace list selects every namespace between it and
  the last-clicked row, inclusive, applying whichever action (add or remove) the last click
  performed - so a mixed-state range follows the anchor's action rather than toggling each row
  individually.
- A shift-click with no prior click in the current editor session ranges from the first listed
  row.

## Capabilities

### Modified Capabilities

- `namespace-sets`: the "Editing a set's namespaces" requirement gains shift-click range
  selection.

## Impact

- `App/app/src/ui/namespace_filter.rs`: the shared filterable namespace list widget (used by both
  the set editor and the title-bar namespace picker) gains range-anchor tracking and a
  shift-aware click handler.
- No change to `App/app/src/ui/namespace_sets/editor.rs` itself - it consumes
  `namespace_filter`'s existing `element_marked`, which already owns click handling.
