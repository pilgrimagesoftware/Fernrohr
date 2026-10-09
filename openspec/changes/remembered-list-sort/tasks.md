# Tasks

## 1. Persistence

- [ ] 1.1 Add `sort_defaults` keyed by group and kind (plus `events`) to `WorkspaceConfig`, storing column ids; verify round-trip tests and that a file without the map loads empty.

## 2. Starting sort and cycle

- [ ] 2.1 Choose each panel's starting sort as own saved sort, then the remembered sort if its column exists, then the kind's default (first default-order column ascending; age ascending for events); verify tests for each branch in the object list, Pods, and events browser panels, including a reordered-columns default and a missing remembered column.
- [ ] 2.2 Replace the unsorted step of the header cycle with the default sort; verify keyboard and click tests of the cycle on a non-default column and on the default column.

## 3. Remembering

- [ ] 3.1 Record a user-initiated sort change (header click or keyboard sort command) into `sort_defaults` through the workspace save path, never on restore, layout load, or default fallback; verify that a new panel of the same kind for another context starts with the recorded sort, and that restoring a layout records nothing.
- [ ] 3.2 Verify end to end that a remembered sort survives a relaunch, using the workspace file round-trip.
