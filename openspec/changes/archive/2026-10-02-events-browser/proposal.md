# Proposal

## Why

When something goes wrong across a namespace, events are where it shows up first, but Fernrohr has
no good way to read them in bulk. Opening the Event kind from the Resource panel gives the generic
list (name, namespace, age), which hides everything that matters - type, reason, the object it is
about, the message - and offers no way to narrow thousands of events down to the ones that matter.

## What Changes

- A dedicated Events browser panel, opened from the Event kind in the Resource panel (replacing its
  generic list) and from a global "Events" command.
- Live table of the cluster's retained events: last seen, type, reason, involved object, message,
  count, source - newest first, both `core/v1` and `events.k8s.io/v1` recorded fields.
- Filters: type (Warning / Normal), namespace (shared with the panel's namespace scope), involved
  object kind, and reason - each a multi-select built from the events present.
- Search: the standard list search box from `resource-list-search`, defaulting to the visible columns
  so the message is searchable.
- An Event's own detail panel gets a real section instead of metadata only.
- The involved object is a link to its detail panel; the selected event's full message is shown
  untruncated in a detail strip.

## Capabilities

### New Capabilities

- `events-browser`: a live, filterable, searchable table of a cluster's retained events.

### Modified Capabilities

- `object-detail`: Event gains a kind-specific section (type, reason, full message, count, times,
  reporter, involved and related objects as links). The Resource panel's routing of the Event kind
  to this browser is an implementation detail of opening the panel.

## Impact

- App: a new events panel beside `object_list`, reusing the shared watch registry (one Event watch
  per context, shared with any other Event consumer), the column/cell machinery from
  `standard-resource-panels`, and the search box from `resource-list-search` when it lands.
- Depends on nothing unmerged; adopts `resource-list-search`'s box if that lands first, otherwise ships
  a name-and-message filter that box later replaces.
