# Design

## Context

Each list panel has an `initial_sort` that is applied when its table is built. The generic object
list and Pods panels default it to `None` (unsorted); the events browser defaults it to
`DEFAULT_SORT` (age ascending). A panel's current sort is already persisted as a `SortState { column,
ascending }` in the workspace file and in saved layouts, and restored into `initial_sort`.
`WorkspaceConfig` already holds one per-context preference map, `namespace_defaults`. See
[proposal.md](proposal.md).

## Goals / Non-Goals

**Goals:**

- Every list table is always sorted, starting from a predictable default.
- A user's sort choice for a kind carries over to the next panel of that kind.

**Non-Goals:**

- Per-context or per-window remembered sorts. One preference per kind keeps it predictable.
- Multi-column sorting.
- Remembering filters, column widths, or column order per kind.

## Decisions

### D1. Per-kind sort defaults in the workspace config

Add `sort_defaults: BTreeMap<String, SortState>` to `WorkspaceConfig`, beside `namespace_defaults`.
The key is the kind's group and kind (`apps/Deployment`, `/Pod` for core, and `events` for the
events browser), not the plural or display name, so it survives discovery differences between
clusters. The column is stored by its column id, not its position, because columns can be
reordered. A file without the map loads with it empty.

### D2. When a sort is remembered

A sort is recorded whenever the user changes it in a panel, by clicking a header or through a
keyboard sort command. Restoring a panel, loading a layout, or falling back to the default never
records anything. Writes go through the existing workspace save path, which is already debounced,
so rapid header clicks cause at most one write.

### D3. Choosing the starting sort

A panel picks its starting sort in this order:

1. Its own saved sort, from session restore or a loaded layout.
2. The remembered sort for its kind, if that column exists in the panel's columns.
3. The default: the first column of the kind's default column order, ascending. For the events
   browser, this is its existing age-ascending default.

The default uses the kind's default column order rather than the user's reordered one. Dragging a
column ahead of Name therefore never changes what "default" means.

### D4. Header cycle without an unsorted state

The header cycle becomes ascending, descending, then default. The third click on a non-default
column applies the default sort, which moves the indicator to the default column. On the default
column itself, the cycle alternates ascending and descending. Returning to the default is a user
choice, so it is remembered like any other (D2), which lets the user reset a kind's remembered sort.

### D5. Keyboard sort commands

Sorting had no keyboard route, which the keyboard-first rule requires. Three commands, in a
`SortableList` key context, provide one: Sort by Next Column (`shift-.`), Sort by Previous Column
(`shift-,`), and Cycle Sort (`o`). Each is a palette entry and rebindable. The header cycle
overrides gpui-component's own default, descending, ascending cycle in each table delegate. The
shared logic is a `SortableTable` trait in `ui::list_sort`, implemented by the object list, Pods,
and events delegates.

## Risks / Trade-offs

- [Users who relied on watch order lose it] -> Watch order is not meaningful to a user, and every
  table now has a stable order instead.
- [A remembered sort on a slow column (such as a computed age) on huge lists] -> Sorting is already
  supported on every column; this only changes which sort a panel starts with.
- [Two windows change the same kind's sort at once] -> Last write wins, which matches how
  `namespace_defaults` behaves.
