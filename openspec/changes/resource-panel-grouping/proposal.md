# Proposal

## Why

The Resource panel is a flat alphabetical list of every kind a cluster's API
discovery reports. On a real cluster that is 60-150 rows of undifferentiated
`SidebarMenuItem`s, and the kinds a user actually wants - Pods, Deployments,
Services, PVCs - are buried in the middle of it among things like
`ComponentStatus` and `TokenReview`. FreeLens and OpenLens both group this list
into named sections; we do not, which is a real gap in a tool whose entire
premise is scanning a cluster quickly.

The list also has no filter. Finding `Ingress` means scrolling and reading every
row, and clusters with heavy CRD installations are long enough that this is
tedious rather than merely annoying.

## What Changes

- Give discovered kinds a **category** - Workloads, Config, Network, Storage,
  Cluster, Access Control, or Custom Resources - and render the Resource panel
  as collapsible sections of one, in a fixed order rather than alphabetically.
- Add a **filter box pinned to the bottom of the Resource panel** that narrows
  the visible rows across every section at once.
- Keep sections' collapsed state per window and out of the preference file, on
  the same reasoning as 11.3's collapse: it is a transient view preference, not
  a durable setting.
- The section headers carry the count of rows they hold, so a collapsed section
  still says how much is inside it.

## Non-goals

- Per-kind filtering *inside* a resource panel's table. Panels already take a
  `filter` in their `ResourceView` (see the `resource-browser` capability); this
  change filters the navigation list, which is a different list with a
  different job.
- Letting users define their own categories, or re-categorise a kind. That is a
  larger feature and nothing here is built to preclude it - see `design.md`.
- Reading CRD annotations to infer categories. Deliberately left out; see
  `design.md` for why, and what it would cost to add.
