# Proposal

## Why

The `resource-browser` spec already requires a text filter on list panels, but it was never
built. The Pods table only scopes by namespace, and `pods/rows.rs` holds filter helpers marked
`// UNWIRED`. A cluster with hundreds of Pods leaves the user scrolling by eye. A plain
name-substring filter also falls short of what people actually look for: they want to keep the
whole list visible and see where the matches are, or match on a status or a label, not only a
name.

## What Changes

- Every panel that lists resources of one kind gets a search box in its header. Pods is the only
  such panel today, and any later list panel adopts the same box. `/` focuses it inside the
  panel. Escape clears it and returns focus to the table.
- A menu attached to the box controls how the search behaves:
  - **Mode**: *Filter* (the default) hides rows that don't match. *Highlight* keeps every row and
    tints the matching ones; Enter and Shift+Enter move the selection to the next or previous
    match.
  - **Scope**: *Name* (the default), *Visible columns* (every cell the table shows), or *Labels*
    (label keys and values).
  - **Match**: *Case sensitive* and *Regular expression* toggles. Both are off by default, which
    gives case-insensitive substring matching.
- The box shows a match count, such as `12 / 340`. An invalid regex marks the box as an error and
  leaves the rows as they were, rather than hiding everything.
- Each panel keeps its own query. The query clears when the panel changes kind. The menu
  settings persist across restarts as the default for new panels.
- Out of scope: searching the full object (spec, env, annotations), highlighting the matched text
  inside cells, and k9s-style label selectors. Searching the full object would also put Secret
  data in reach of a search.

## Capabilities

### New Capabilities

### Modified Capabilities
- `resource-browser`: the "Filtering and sorting" text filter becomes a header search box with
  filter/highlight modes, a selectable scope, case/regex options, a match count and persisted
  settings.

## Impact

- New shared search component in the App (header input, options menu, matcher) built on
  `gpui_kit`'s `Input`/`InputState` and `DropdownMenu`/`PopupMenu`, so other list panels can
  reuse it.
- `app/src/k8s/resource/pods/` (`render.rs`, `rows.rs`, `panel.rs`) and `pods_table.rs`: wire the
  matcher into the row pipeline, tint matched rows in Highlight mode, and replace the unwired
  `matches_filter`.
- A new command registered in the Pods panel's key context, bound to `/` by default. It doesn't
  clash with the Resource panel's existing `/`, which lives in that panel's own key context.
- `ui.toml` gains the persisted search defaults.
- The `regex` crate, if it isn't already a dependency.
