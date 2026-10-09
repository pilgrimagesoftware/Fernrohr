# Design

## Context

`pod_detail/field_view.rs`'s `render_field` matches on `PodFieldValue` to build each tab's
content, then every arm except `Containers` falls through to the shared
`detail::row(field.label, value, cx)` wrapper at the end of the function, which draws the
field's label to the left of its value. For most tabs that's correct - Overview has many
labeled fields in one view. But the Managed Fields tab has exactly one field
(`PodFieldValue::ManagedFields`, labeled `"Managed Fields"`) filling the whole tab, so the row
wrapper duplicates the tab heading for no reason. `Containers` already hit this same problem and
solved it by returning early, bypassing the wrapper.

## Goals / Non-Goals

**Goals:**
- Remove the redundant label from the Managed Fields tab's content area.
- Keep every other tab's labeling (including Managed Fields' own disclosure-row summaries)
  unchanged.

**Non-Goals:**
- Changing the Managed Fields tab's heading, its disclosure-row layout, or the "Tab keys follow
  tab order" hint row in `render.rs` (which already just shows `section.label()` next to the key
  hint, not the content-area label this change removes).

## Decisions

- **Early-return from the `ManagedFields` arm, mirroring `Containers`.** Change
  `PodFieldValue::ManagedFields(entries) => self.render_managed_fields(entries, cx),` to return
  directly (`return self.render_managed_fields(entries, cx);`) instead of falling through to
  `detail::row`. This is the same pattern already used one arm up for `Containers`, so the fix is
  consistent with existing precedent rather than a one-off special case.

## Risks / Trade-offs

- None of note - this narrows one tab's rendering to drop one label; no behavior, state, or
  keyboard path changes.
