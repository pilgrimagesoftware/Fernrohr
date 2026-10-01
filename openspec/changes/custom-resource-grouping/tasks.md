# Tasks

## 1. Grouped picker data model

- [ ] 1.1 Add a view-model step that takes `discover_kinds`'s already-sorted output and buckets it
      into `(ApiGroup, Vec<DiscoveredKind>)` sections (core first, others alphabetical), and
      verify with a unit test asserting section order and membership for a mixed core/CRD input
- [ ] 1.2 Add per-cluster-connection session state (`HashSet<ApiGroup>` of collapsed groups,
      starting empty) and verify with a unit test that a fresh connection starts with every group
      expanded

## 2. Sectioned rendering

- [ ] 2.1 Render the picker as sections with group headers, hiding a section's kinds when its
      group is collapsed, and verify with a UI test that collapsing a group hides its kinds while
      leaving other sections visible
- [ ] 2.2 Wire a click/keybinding on a group header to toggle that group's collapsed state in the
      session set, and verify with a test that toggling updates both the rendered state and the
      session's collapsed set

## 3. Filtering across groups

- [ ] 3.1 Change the existing text filter to compute matches per-kind first, then derive which
      groups have at least one match, and verify with a unit test that a filter matching kinds in
      two of three groups yields exactly those two groups' sections
- [ ] 3.2 Make a matching collapsed group render expanded while a filter is active, and restore
      its prior collapsed state when the filter is cleared, and verify with a test covering
      filter-applied, then filter-cleared, with the original collapsed state intact

## 4. Keyboard navigation

- [ ] 4.1 Flatten the grouped-and-filtered view into one focus-order list (headers and visible
      kinds) and route arrow-key/type-ahead navigation through it, and verify with a test that
      pressing down-arrow at a group's last visible kind moves focus to the next section's first
      item
- [ ] 4.2 Add a "toggle group" action bound to a key while a kind or header has focus, moving
      focus to the header when it collapses, and verify with a test asserting both the collapse
      and the focus move

## 5. Manual verification

- [ ] 5.1 Open the picker against a cluster (or fixture) with multiple CRD groups and confirm
      visually that grouping, collapse/expand, filtering, and keyboard navigation behave as
      specified; record the result in this change's tasks.md
