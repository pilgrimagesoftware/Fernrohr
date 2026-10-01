# Tasks

## 1. Subgroup data model

- [x] 1.1 Bucket the Custom Resources section's kinds into `(api_group, Vec<DiscoveredKind>)`
      subgroups (core first, others alphabetical, existing kind order within each), and verify
      with a unit test asserting subgroup order and membership for a mixed input
- [x] 1.2 Add a per-window `HashSet<String>` of collapsed subgroups next to the existing
      `HashSet<Category>`, starting empty, and verify with a unit test that a new window starts
      with every subgroup expanded

## 2. Subgroup rendering

- [x] 2.1 Render subgroup headers inside Custom Resources, hiding a subgroup's kinds when
      collapsed, and verify with a UI test that collapsing one subgroup leaves others visible
- [x] 2.2 Wire a click on a subgroup header to toggle it in the collapsed set, and verify with a
      test that toggling updates both rendered state and the set

## 3. Filtering across subgroups

- [x] 3.1 Compute filter matches per-kind, then derive which subgroups have a match, and verify
      with a unit test that a filter matching kinds in two of three subgroups yields exactly those
      two
- [x] 3.2 Force a matching collapsed subgroup open while a filter is active and restore its prior
      state when cleared, and verify with a test covering filter-applied then filter-cleared

## 4. Keyboard navigation

- [x] 4.1 Add subgroup headers to the panel's flattened focus order, and verify with a test that
      down-arrow at a subgroup's last visible kind moves focus to the next subgroup's header
- [x] 4.2 Add a "toggle group" action through the existing command/keybinding system, moving focus
      to the header on collapse, and verify with a test asserting both the collapse and focus move

## 5. Manual verification

- [x] 5.1 Open the Resource panel against a cluster (or fixture) with multiple CRD groups and
      confirm visually that subgrouping, collapse/expand, filtering, and keyboard navigation
      behave as specified; record the result here - passed 2026-10-01 on macOS against a
      cluster with several CRD groups: subgroups ordered alphabetically under Custom Resources,
      click/Space collapse, filter forces matches open and restores on clear, arrow keys step
      through headers

## 6. Collapsed by default, collapse/expand all

- [x] 6.1 Start every Custom Resources subgroup collapsed in a new window (including groups that
      appear later via discovery), keeping state per window, and verify with a unit test that a
      fresh panel reports every subgroup collapsed and that expanding one leaves the rest collapsed
- [x] 6.2 Add "Collapse All Resource Groups" and "Expand All Resource Groups" commands through the
      existing command/keybinding system, scoped like the toggle (not while typing in the filter),
      shown in the hint row, and verify with tests that each sets every subgroup's state and that a
      collapse-all moves a focused kind's cursor to its subgroup header
