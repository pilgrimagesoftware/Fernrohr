# Tasks

## 1. Live pod detail

- [x] 1.1 Pod detail subscribes to the context's shared Pods watch on open (unsubscribe on close), reads its pod from the table, and re-renders on changes; keep the one-shot get only for first paint before the table syncs. Verify with a mock-server window test: a Pending pod becomes Running and the panel updates without reopening; closing releases the subscription.
- [x] 1.2 Preserve view state across updates (tab, scroll, revealed Secret, expanded managed fields, folded YAML). Verify a restart-count change keeps the open tab and an expanded row.
- [ ] 1.3 Deletion live: Terminating (deletion timestamp, grace countdown), then "deleted at <time>" with last known fields kept and marked stale, Events tab still listing the pod's events, panel stays open; a same-name pod with a new uid replaces it with a visible notice. Verify with mock-server window tests for delete-while-open and StatefulSet-style recreate.

## 2. Live object detail

- [ ] 2.1 Object detail does the same through the kind's shared `ObjectsTable` (polling kinds included), including delete-while-open (deleted at <time>, last state kept as stale). Verify with tests for a Deployment's replica counts updating live and for its deletion.

## 3. Verification

- [ ] 3.1 `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` and `cargo test` pass.
- [ ] 3.2 Manual check: the user's repro - a pod waiting on a missing Secret goes Running in its open detail panel once the Secret exists; then delete the pod and confirm the panel shows Terminating, then deleted with its last state and events kept.

## Notes

- Implemented in App#112. Object detail deviates from D1 on purpose: `ObjectsTable` rows hold list cells, not whole objects, so a detail panel uses its row's `resourceVersion`/uid as the change signal and refetches through the redacting `get` - one GET per write to that object, with Secret values never in the shared table.
