# Design

## Context

See proposal.md. Pieces to reuse: the shared watch registry and `watch_stream::run`
(`standard-resource-panels` D3), the `Cell`/column machinery (D2), `events.rs::event_time` for
`events.k8s.io/v1` fields, `OpenListedObject` / `NavTarget::Object` for links, and
`resource-list-search`'s search box (proposed, not built).

## Goals / Non-Goals

**Goals:** one live events table per context with facet filters, search, and links.

**Non-Goals:** keeping events past the cluster's retention; aggregating repeated events across
objects; an events timeline chart.

## Decisions

### D1: Typed Event watch through the shared registry
A `WatchKey::Events` entry in the session's registry, watching `core/v1` `Event` across all
namespaces (or the selected ones), so two events browsers on one context share one watch. Typed
rather than `DynamicObject` because every column needs typed fields and `event_time`.

### D2: Facets computed from the current rows
Filter options are the distinct values in the retained events, recomputed on watch events. Filters
are applied before search; both client-side. Filter state is part of the panel's saved state.

### D3: Search adopts `resource-list-search`
If that change has landed, the panel uses its box with scope defaulting to visible columns. If not,
this change ships a minimal case-insensitive substring filter over reason, object and message, built
behind the same `/` key so the later box replaces it without changing keys.

### D4: Routing
`NavTarget::Kind` for core `Event` routes to the events browser instead of `ObjectListPanel`; the
"Events" command opens it for the window's active context.

## Risks / Trade-offs

- [Busy clusters retain many thousands of events] -> the table already virtualizes rows; facets
  and filters are computed once per watch batch, not per frame.
- [Overlap with `resource-list-search`] -> D3's interim filter is deliberately minimal.
