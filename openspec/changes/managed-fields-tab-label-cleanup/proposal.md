# Proposal

## Why

The Managed Fields tab shows a "Managed Fields" label to the left of its content, duplicating
the tab's own name - the tab is already titled Managed Fields, so the content-area label adds
no information and just takes up width other tabs don't spend on themselves.

## What Changes

- Remove the "Managed Fields" label rendered beside the Managed Fields tab's content, since the
  tab heading already says what the content is.

## Capabilities

### New Capabilities

(none)

### Modified Capabilities

- `pod-detail`: clarifies that the Managed Fields tab's content area shows only the disclosure
  rows, with no redundant section label.

## Impact

- `App/app/src/k8s/resource/pod_detail/field_view.rs`: `render_field`'s `PodFieldValue::ManagedFields`
  arm currently falls through to the shared `detail::row(field.label, value, cx)` wrapper at the
  end of the function, which draws the "Managed Fields" label beside the content. Change it to
  early-return the rendered body directly, the same way the `PodFieldValue::Containers` arm
  already skips that wrapper for its own full-width content.
