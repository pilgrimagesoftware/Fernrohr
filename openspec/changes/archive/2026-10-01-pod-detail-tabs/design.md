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

The first cut used three tabs (Overview, Containers, Conditions). Using it showed that grouping
was wrong in two ways: Volumes and Managed Fields were each large enough to bury what shared a
tab with them, and the view had no Events at all - the first thing a reader reaches for when a
pod is misbehaving. The revised grouping, left to right:

- **Overview**: Created, Name, Namespace, Labels, Annotations, Controlled By, Status, Node,
  Host IPs, Pod IPs, Service Account, QoS Class, Termination Grace Period, Tolerations,
  Conditions. Conditions and Tolerations are short once they are not competing with container
  cards, and read naturally beside Status and Node.
- **Containers**: Containers, Init Containers.
- **Volumes**: Volumes, always shown (`PodFieldValue::List`) rather than behind a Show/Hide - the
  tab exists to show them, so a disclosure would only be an extra click.
- **Events**: the events naming this pod, newest first (see below). Not a `PodField`: events are
  not part of the `Pod` object `pod_fields` projects.
- **Managed Fields**: one block per manager (name and operation), each independently expandable
  to its pretty-printed `fieldsV1` ownership tree. Rightmost, as the tab read least often - it is
  about who wrote a field, not what the pod is doing.

## Events

Fetched alongside the pod in the same `fetch_pod` call rather than a second fetch lifecycle - the
tab has nothing to show until the pod has loaded anyway, and a 404 on the pod skips the lookup.

- The field selector pins `involvedObject.kind=Pod` and, when present, `involvedObject.uid` as
  well as namespace and name: a Service can share a pod's name, and a StatefulSet pod is recreated
  under the same name, so name alone would mix in another object's (or a predecessor's) events.
- A failed events list does not fail the panel: the pod still loads, and the Events tab says why
  it has nothing to show rather than claiming "No events." - `get` on pods and `list` on events
  are separate RBAC grants.
- Timestamps and counts read the `events.k8s.io/v1` fields (`series.lastObservedTime`,
  `eventTime`, `series.count`) when the legacy `lastTimestamp`/`count` are absent, as they are on
  scheduler-written events.

## Which tab component

`gpui-component` likely has a `Tabs`/`TabBar` primitive already in this dependency tree (the dock
itself uses tabs internally - `dock::tab_panel`). Reuse that rather than building a second tab
implementation; confirm its public API covers a plain content-switching use (not just the dock's
panel-management one) before committing to it in tasks.md.
