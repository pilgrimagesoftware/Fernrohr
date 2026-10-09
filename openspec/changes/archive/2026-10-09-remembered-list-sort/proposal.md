# Proposal

## Why

A newly opened list panel starts unsorted, so rows appear in whatever order the watch delivers them,
and a sort the user picked for one Deployments panel is forgotten the moment they open the next
one. Users re-sort the same kinds by the same column over and over.

## What Changes

- A list panel with no saved sort opens sorted by the kind's first column (usually Name),
  ascending, instead of unsorted.
- The application remembers the last sort the user chose for each resource kind, across contexts,
  windows, and restarts. A newly opened panel of that kind starts with it.
- A panel that already has its own sort, restored from the last session or from a saved layout,
  keeps that sort. A remembered sort only seeds new panels.
- The header click cycle becomes ascending, descending, then back to the default sort, so a table
  is never left in watch order.
- The events browser keeps its own default (newest first) and also remembers the user's choice.

## Capabilities

### New Capabilities

- None.

### Modified Capabilities

- `resource-browser`: Filtering and sorting changes the default sort and the header click cycle,
  and a new requirement covers remembered per-kind sorts.
- `events-browser`: the events browser remembers the user's chosen sort for new events browsers.

## Impact

- `config/workspace.rs` (`WorkspaceConfig` gains per-kind sort defaults beside its namespace
  defaults) and its persistence.
- Generic object list, Pods, and events browser panels: how `initial_sort` is chosen and where a
  sort change is recorded.
- No change to saved layouts or session restore formats; both already store each panel's sort.
