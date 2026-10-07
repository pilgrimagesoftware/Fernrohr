# Tasks

## 1. Shared condition-message derivation

- [ ] 1.1 Add `condition_message` (or equivalent) to `app/src/k8s/resource/status_tone.rs`, reading
      a raw `serde_json::Value` conditions array: prefers `type == "Ready"` (Good/Bad/Warning by
      `status`), else the most recently transitioned condition with a non-empty message (untoned);
      `None` when nothing qualifies. Verify with unit tests in `status_tone/tests.rs` covering: Ready
      True/False/Unknown, no Ready condition (fallback by latest `lastTransitionTime`), empty
      message, malformed/missing `lastTransitionTime`, and empty conditions array.
- [ ] 1.2 Generalize `object_detail/sections/common.rs::condition_badges` callers (not its
      signature, which already accepts `impl IntoIterator<Item = (&str, &str)>`) so a JSON-sourced
      `(type, status)` iterator can be built from raw conditions; verify with a unit test that feeds
      it conditions parsed from JSON rather than a typed struct's fields.

## 2. List column: generic Message for kinds outside the table

- [ ] 2.1 Extend the list panel's column resolution so that a kind with no `for_kind` entry adds a
      one-column `KindColumns`-equivalent (Message) once the panel's live store has observed an
      object of that kind with a non-empty `status.conditions`; verify with a test that opens a
      panel for a fixture CRD kind with no conditions (no Message column), then delivers a watch
      event with conditions (Message column appears).
- [ ] 2.2 Wire the Message cell to `condition_message`, producing `Cell::Status(text, tone)` for a
      toned Ready message and `Cell::Status(text, Tone::Neutral)` (or `Cell::text`, untoned) for the
      fallback case; verify with unit tests covering the toning matrix (Good/Bad/Warning/untoned).
- [ ] 2.3 Confirm no built-in kind already in `for_kind`'s table also gains a Message column;
      verify with a test asserting the Jobs (or another already-covered kind's) column set is
      unchanged.
- [ ] 2.4 Verify sort, resize and reorder work on the new column the same as any other, by
      extending the existing per-kind-columns table test suite with one CRD-fixture case.

## 3. Object detail: message field for kinds with existing sections

- [ ] 3.1 Add a `"Message"` `ObjectField` (via `FieldValue::Status`) immediately before the existing
      `"Conditions"` field in each workload section that already calls `condition_badges`
      (Deployment, DaemonSet, StatefulSet in `sections/workloads.rs`), sourced from the same typed
      object's `status.conditions` through `condition_message`; verify with updates to
      `object_detail/tests/workloads.rs` asserting both the Message field and the unchanged
      Conditions badges are present.
- [ ] 3.2 Confirm an object of one of these kinds with no qualifying condition message shows no
      Message field while keeping its Conditions badges; verify with a fixture test for that case.

## 4. Object detail: generic Status section for kinds with no dedicated section

- [ ] 4.1 Add a generic fallback branch to `object_detail/sections/mod.rs::sections_for`'s `_ =>
      None` arm: when the raw object has a non-empty `status.conditions` array, build one
      `ObjectSection` titled "Status" with a `"Message"` field and a `"Conditions"` badges field
      (reusing the generalized `condition_badges` caller from 1.2); otherwise keep returning `None`
      (metadata only, unchanged). Verify with a new `object_detail/tests/sections.rs` (or a
      dedicated fixture) case using a synthetic CRD object shaped like a Flux `Kustomization` or
      `HelmRelease`, asserting the Status section appears with both fields.
- [ ] 4.2 Verify a CRD object with no `status.conditions` still gets metadata only, unchanged from
      current behavior, with a regression test alongside 4.1's.

## 5. Copy and keyboard access

- [ ] 5.1 Extend the detail panel's copy-control wiring so the derived Message field is copyable on
      hover and by keyboard, per the modified "Values can be copied" requirement; verify with a test
      asserting the message text lands on the clipboard, mirroring the existing container-image copy
      test.
- [ ] 5.2 Verify the list panel's existing describe (`d`) keyboard route opens the object's detail
      panel from a row with a truncated Message cell, surfacing the full text there; verify with a
      keyboard-simulated test (`VisualTestContext::simulate_keystrokes`) extending the existing
      describe-from-list coverage with a conditions-bearing CRD fixture.

## 6. Integration check

- [ ] 6.1 Run `cargo fmt -- --check`, `cargo clippy -- -D warnings`, and `cargo test` in `App/` and
      confirm all new and existing tests pass.
- [ ] 6.2 Manually verify against a cluster with Flux installed (or recorded fixtures standing in
      for one): open a `HelmRelease` or `Kustomization` list and detail panel and confirm the
      Message column and Status section both show the Ready condition's message with the expected
      tone, with no cluster-identifying details recorded in any verification notes.
