# Tasks

## 1. Category model

- [ ] 1.1 Add a `Category` enum carrying the seven sections in their fixed
  display order, and a lookup from `(api group, plural)` to a category that
  returns `Custom Resources` for anything unmatched. Verify: a test maps core-group
  `Pod`, `Service`, `ConfigMap`, `PersistentVolumeClaim`, `ServiceAccount` and
  `Namespace` and asserts six different categories, and asserts an unknown
  group/plural pair returns `Custom Resources`.
- [ ] 1.2 Populate the table for the built-in Kubernetes kinds across all seven
  sections, transcribed from the Kubernetes API reference's own category
  taxonomy. Verify: a test asserts each section is reachable by at least one
  built-in kind, and that no built-in kind is unmapped - a `Custom Resources`
  bucket that catches `Deployment` is the failure this guards.

## 2. Grouped rendering

- [ ] 2.1 Partition the discovered kinds into sections, dropping empty ones and
  ordering the rest by the fixed order. Verify: a test feeds a kind set spanning
  several sections plus a duplicate-free CRD, and asserts the resulting section
  order and that no kind is lost.
- [ ] 2.2 Render each section as a collapsible header showing the section name
  and its kind count, over its rows. Verify: a test collapses a section and
  asserts its rows are hidden while its header and count remain.
- [ ] 2.3 Keep collapse state in the `ResourcePanel`, default expanded, and do
  not write it to the preference file. Verify: a test collapses a section, then
  asserts a freshly constructed panel has every section expanded.

## 3. Filter

- [ ] 3.1 Add a filter field pinned to the panel's bottom edge, with
  case-insensitive substring matching over a row's label, kind, plural and API
  group. Verify: a test matches on each of the four fields in turn, including a
  group name that appears in neither the label nor the kind.
- [ ] 3.2 While a filter is active, render sections with matches expanded
  regardless of collapse state, and hide sections with none - without mutating
  the stored collapse state. Verify: a test collapses a section, filters to a
  kind inside it and asserts it is visible; then clears the filter and asserts
  the section is collapsed again.
- [ ] 3.3 Report "nothing matched" when the filter matches no kind. Verify: a
  test filtering on a substring no kind contains asserts the panel renders no
  sections and shows the empty-result message.

## 4. Keyboard

Required by `.claude/rules/keyboard-first.md`: every action above has a keyboard route as
well as a mouse one, with one selection that clicks and keys both move (never hover).

- [ ] 4.1 Make the list keyboard-navigable: with the panel focused, Up/Down move the
  selection across visible rows (skipping collapsed sections' rows), Enter opens the
  selected kind exactly as a double-click does, and Left/Right collapse/expand the
  selected row's section. A click selects the same way. Verify: tests using real
  keystrokes (`simulate_keystrokes`) for Down, Enter, Left and Right, and that hover does
  not move the selection.
- [ ] 4.2 Add a filter-focus command (`/` in the Resource panel's `KeyContext`, registered
  in the command registry) and Escape in the filter to clear it and return focus to the
  list. Verify: a keystroke test for `/`, typing, and Escape.
- [ ] 4.3 Show the panel's keys in a hint row read from the live keymap
  (`Kbd::binding_for_action`). Verify: a render check that the hint row names the bound
  keys.
- [ ] 4.4 Manual check: navigate, open, collapse/expand and filter the Resource panel on a
  real QA cluster using only the keyboard, then the same with only the mouse.

