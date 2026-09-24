---
paths:
  - "**/*.rs"
  - "crates/**"
  - "App/**/*.rs"
---

# Rust structure and maintainability conventions

These rules are adopted from [pilgrimagesoftware/Knot](https://github.com/pilgrimagesoftware/Knot/blob/develop/.claude/rules/rust-structure.md).
Each exists because it was violated and cost something; the cost is named so you can tell when the rule genuinely does not apply.

## File size: 700 lines, enforced

Do not raise any equivalent line limit to make a change fit. Split files by concern, not by line count:

- Pull out the group of items that answer one question.
- Give the new module a doc comment saying what it owns and what it does not.
- An inherent `impl` can be split across files — `impl Foo { .. }` may appear in several modules of the same crate. A large `impl` therefore splits with no call site changing.
- Keep the parent's public surface identical by re-exporting (`pub(crate) use submodule::*;`). Which file an item lives in is the module's business, not its callers'.

Colocated `#[cfg(test)] mod tests` counts toward the limit. Move it to a sibling `tests.rs` before splitting production code that is fine as it is.

## Visibility: widen only as far as the move requires

When splitting, a private item used by a sibling becomes `pub(super)`, or `pub(in crate::some::module)` when the module nests deeper. It does not become `pub(crate)` "to be safe", and it never becomes `pub`.

A visibility wider than the code needs is a claim about who may depend on the item, and it is the claim — not the keyword — that costs later.

## No crate-wide `allow`

If something must be allowed, allow it on the item, with a comment saying why:

```rust
// UNWIRED(#222): feature X's decision layer. Nothing calls `function_name`,
// so this is reached only from tests.
#[allow(dead_code)]
pub(crate) fn function_name(..) -> bool { .. }
```

Markers in use, both greppable:

- `UNWIRED` — ported from spec, no caller yet. Reference the tracking issue when there is one.
- `SUPERSEDED` — a newer path replaced this; it is waiting to be deleted.

A test that exercises unwired code is not coverage. Say so in the test module's doc comment.

## Constants live in `consts.rs`

One per crate. A value belongs there when it is a _decision_ — how often to poll, how stale a cache may get, what colour a state is. A value stays inline when it is part of one element's layout: a padding, a gap, a single width.

Constants that are only meaningful together at one call site may stay local, named and documented in place; hoisting those makes them harder to find, not easier.

## No I/O on hot paths

Render paths, event handlers, and tight loops must not perform blocking I/O (subprocess calls, file reads, or blocking locks). Cache I/O results and refresh asynchronously.

When I/O must live in a hot path, document the trade-off and mark it with a `ponytail:` comment naming the ceiling and upgrade path.

## Off-thread results must reach a frame

Work that finishes off the main thread reports itself through a flag — a dirty bit, a `RefreshCache`, an `Arc<AtomicBool>` that `spawn_blocking` sets. The main thread polls those flags and calls `repaint` or equivalent. Every link in that chain has now been broken at least once across projects:

- **The flag nobody polls.** A dirty bit is set but never read.
- **The flag read and then discarded.** A clearing read inside a branch that discards it.
- **The work reported to nobody.** Async work completes but never notifies the UI.
- **The claim that never records.** A cache is marked as "refresh requested" but the result is never stored.

What to check:

1. A new flag, cache or `spawn_blocking` result must appear in the polling/repaint chain.
2. A clearing read must not be reachable on a path that discards it.
3. Assign clearing reads to locals before combining them. `a.take() || b.take()` skips the second whenever the first is true.
4. Between the "request refresh" call and the work being sent off, there must be no early return. Hand it to `spawn_blocking` on the next line.

## Locks: `parking_lot`, and a guard that does not outlive its statement

Use `parking_lot::Mutex` for synchronous locks — it has no poisoning and `lock()` returns the guard directly.

`tokio::sync::Mutex` stays in exactly one place: where the guard is held across an `.await`. Everywhere else, the `parking_lot` guard is `!Send`, which is what makes the compiler reject holding it across an await.

Scope a lock so it is dropped before any blocking call — take it once around a loop rather than once per iteration, or another thread can observe inconsistent state.

## Owning spawned work

A `tokio::spawn` whose `JoinHandle` is dropped cannot be cancelled and swallows its panic. Store the handle and abort it in `Drop`.

A background task must never hold a strong `Arc` back to the thing that owns its lifetime. Use a `Weak` and upgrade per iteration.

## Parallel per-key maps

If a struct holds multiple `BTreeMap<Uuid, _>` or similar that must all be pruned together, prefer one struct per key over N maps keyed alike. Where the maps already exist, teardown belongs in one function and every new field must be added to it in the same commit.

## `mod.rs` declares; it does not implement

A `mod.rs` holds module declarations, re-exports, and the doc comment saying what the module owns. Implementation goes in sibling files named for what they do — `foo/handler.rs` for message handling, `foo/state.rs` for state management.
