# Design

## Context

The object list, Pods, and events browser panels are backed by stores that apply kube watcher
events. A relist arrives as `Init`, then any number of `InitApply`, then `InitDone`. The events
browser store already tracks the relist to sweep stale rows. Today no store exposes whether a list
has completed, so an empty table is ambiguous. See [proposal.md](proposal.md).

## Goals / Non-Goals

**Goals:**

- Distinguish loading, empty, filtered-to-nothing, and loaded.
- Never blank a table that already has rows.

**Non-Goals:**

- Progress percentages (the API does not report a total), or loading indicators for detail
  panels.

## Decisions

### D1. Load phase in each store

Each store tracks a `LoadPhase`: `Loading { received }` from creation or `Init` until `InitDone`,
then `Loaded`. `received` counts `InitApply` events in the current relist. A store that has
completed at least once is in a refresh when it sees `Init` again. The shared logic lives in one
small type used by all three stores, so their behavior cannot drift apart.

### D2. Rendering

- First load (never loaded): the table body shows a centered spinner and "Loading <Kind>…", with
  "<n> received" once `received` > 0.
- Loaded with no rows: "No <Kind> in <scope>", or "No <Kind>" for cluster-scoped kinds. Loaded with
  rows that the filter hides: "No rows match the filter".
- Refresh with rows on screen: the rows stay, and a small spinner with the tooltip "Refreshing"
  shows in the panel header beside the namespace selector.

Rows that arrive during the first load are buffered until `InitDone`, as today, so the table never
renders half a list in a misleading order.

### D3. Delay

Both indicators appear only after the phase has lasted 300 ms, using a timer task that redraws the
panel. A load that finishes sooner draws nothing.

### D4. Spinner

Use gpui-kit's spinner or indicator component if it has one. Otherwise, use a small rotating icon
from the existing icon set. Either way, it uses the theme's muted foreground color.

## Risks / Trade-offs

- [A relist that never completes shows the indicator forever] -> The existing watch failure
  handling (refusals and connection pauses) still applies, and the status bar shows the
  connection's state.
