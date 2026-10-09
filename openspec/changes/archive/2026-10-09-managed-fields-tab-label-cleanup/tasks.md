# Tasks

## 1. Remove the redundant label

- [ ] 1.1 In `App/app/src/k8s/resource/pod_detail/field_view.rs`'s `render_field`, change the
  `PodFieldValue::ManagedFields(entries) => self.render_managed_fields(entries, cx),` arm to
  `return self.render_managed_fields(entries, cx);`, matching the early return already used by
  the `Containers` arm just above it, so the match's final `detail::row(field.label, value, cx)`
  is skipped for this field. Verify with `cargo build`.
- [ ] 1.2 Update or add a test asserting the Managed Fields tab's rendered output contains no
  "Managed Fields" label element beside its disclosure rows, while the disclosure rows themselves
  still render as before. Verify with `cargo test`.
- [ ] 1.3 Run the app (`cargo run`), open a pod detail panel, switch to the Managed Fields tab,
  and visually confirm the label is gone and the disclosure rows are unaffected.
- [ ] 1.4 Run `cargo fmt && cargo clippy -- -D warnings && cargo test` and confirm all pass.
