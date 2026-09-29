# Design

## Tabs group existing fields; they don't change what a field is

`pod_fields(&Pod, Timestamp) -> Vec<PodField>` stays the single source of truth for content - the
tabs only decide which of those `PodField`s render together. The cleanest way to keep this true
rather than aspirational: give `PodField` a `section: DetailSection` (an enum: `Overview`,
`Containers`, `Conditions`), assigned at push-time in `pod_fields` the same way `label` already
is, and have the render side group by it. This keeps the "every field the design lists is
projected in order" tests meaningful (they still assert on the flat list) while giving the render
layer what it needs to partition without re-deriving section membership from the label string.

## Which fields go where

- **Overview**: Created, Name, Namespace, Labels, Annotations, Controlled By, Managed Fields,
  Status, Node, Host IPs, Pod IPs, Service Account, QoS Class, Termination Grace Period.
- **Containers**: Containers, Init Containers, Volumes (volumes belong next to what mounts them,
  not in Overview).
- **Conditions**: Conditions, Tolerations (scheduling-related, reads naturally next to
  conditions rather than buried in Overview).

This is a starting grouping, not a mandate - worth a second look once tabs exist and the panel can
actually be used, since "does this field belong here" is easier to judge looking at the real
layout than reasoning about it in the abstract.

## Which tab component

`gpui-component` likely has a `Tabs`/`TabBar` primitive already in this dependency tree (the dock
itself uses tabs internally - `dock::tab_panel`). Reuse that rather than building a second tab
implementation; confirm its public API covers a plain content-switching use (not just the dock's
panel-management one) before committing to it in tasks.md.
