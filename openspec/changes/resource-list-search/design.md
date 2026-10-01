# Design

## Context

See proposal.md for the motivation. The facts that shape the approach:

- The Pods table (`k8s/resource/pods_table.rs`, `PodTableDelegate` on `gpui_kit`'s
  `TableDelegate`) is the only per-kind list panel today. `PodsPanel::render`
  (`pods/render.rs`) builds `PodTableRow`s from the watched Pods, applying only
  `matches_namespaces`, and hands them to `sync_table`.
- `pods/rows.rs` has a pure `view_rows` pipeline (namespace → row → name filter → sort) that
  `render` doesn't call, with `matches_filter` marked `// UNWIRED`.
- The Resource panel (`ui/panel/resource.rs`) already has a working filter: an `Input` bound to
  an `InputState`, a `resource.focus_filter` command bound to `/` in the `ResourcePanel` key
  context, and Escape to clear. That is the convention to match.
- Nothing in the app highlights search matches today. Row tinting reuses the existing
  whole-row styling approach (the selected/accent row background), not text spans.
- `ui.toml` (`config/ui.rs`, `UiConfig`) is the persisted preferences file, with the typed
  `load()`/`save()` pattern from `docs/architecture.md`.

## Goals / Non-Goals

**Goals:**
- One search component that any list panel adopts by supplying its rows' searchable values,
  with Pods as the first user.
- Matching logic that is pure and GPUI-free, so the scope, case, regex and invalid-regex rules
  are unit-tested without a window.

**Non-Goals:**
- Retrofitting the Resource panel's kind filter onto this component. It filters sections, not
  table rows, and already meets its own spec.
- The Logs panel filter (`logs-search-filter`). It may reuse the matcher later, but it isn't
  touched here.
- Building list panels for kinds other than Pods.

## Decisions

### A `ListSearch` entity plus a pure `SearchQuery` matcher

`ListSearch` is a GPUI entity owning the `InputState`, the menu settings and the current query.
It renders the header box, menu and count. `SearchQuery` is a plain struct built from text plus
settings (`mode`, `scope`, `case_sensitive`, `regex`), compiled once per text or settings change
into either a lowered substring or a `regex::Regex`, or an `Invalid` marker. It exposes
`matches(&dyn Searchable) -> bool`.

*Alternative:* put the matching inside `PodTableDelegate`. Rejected, because every future list
panel would copy it, and the delegate is GPUI-bound and harder to test.

### Panels describe their rows through a `Searchable` trait

`trait Searchable { fn name(&self) -> &str; fn visible_values(&self, columns: &[ColumnId]) ->
Vec<Cow<str>>; fn labels(&self) -> &BTreeMap<String, String>; }`. `PodTableRow` implements it.
Visible columns come from the table's current column order and visibility, so a hidden column is
never matched. Labels are matched as `key=value` strings.

This keeps the scope rules (spec: "List search scope") in one place, and it can't reach Secret
data or a spec, because the trait never exposes them.

### Filtering before the table, tinting inside it

Filter mode drops non-matching rows in `PodsPanel::render` before `sync_table`, the same place
namespace scoping happens today. It replaces the unwired `view_rows`/`matches_filter`; delete
those rather than wiring them up, since `sync_table` already owns sorting. Highlight mode passes
every row through with a `matched: bool` on `PodTableRow`, and `render_tr`/`render_td` applies a
tint drawn from the theme's accent at low alpha, so it stays distinct from the selected-row
background. The count is computed in the same pass.

### Next/previous match moves the table selection

Enter and Shift+Enter in the box move `TableState`'s selected row to the next or previous
`matched` row after the current selection, wrapping around, using the visible (sorted) order. This
reuses the existing selection path, so the `SelectedPod` global and the detail panel follow it
automatically.

### Keybinding

There's a new `pods.focus_search` command, bound to `/` by default in the Pods panel's key context
(`PANEL_KEY_CONTEXT`), registered through the panel's `register_commands`. Escape is handled in
the input, as the Resource panel does.

### Persistence

`UiConfig` gains a `list_search: ListSearchDefaults { mode, scope, case_sensitive, regex }` with
`#[serde(default)]`, so existing `ui.toml` files load unchanged. A new `ListSearch` reads it at
construction. Changing a setting in the menu saves it, but doesn't propagate to panels that are
already open (spec: "List search state and persistence").

### Regex dependency

Add `regex` as a direct dependency. It's already in `Cargo.lock` transitively, so it adds nothing
to the build. Compile on text change, not per row.

## Risks / Trade-offs

- [Visible-columns matching formats every cell on each keystroke] → Rows are already formatted for
  display, so the trait borrows those strings. Pod counts in the thousands stay well under a
  frame. Revisit with caching only if profiling shows otherwise.
- [A pathological regex is slow] → The `regex` crate guarantees linear time. Compile errors become
  the Invalid state.
- [The tint is too close to the selection colour in some themes] → Derive it from the accent at a
  fixed low alpha, and check both light and dark themes in the smoke test.
- [`/` inside the search box types a slash] → The command is bound in the panel context. When the
  input has focus, the input's own key handling wins, so `/` is just typed.
