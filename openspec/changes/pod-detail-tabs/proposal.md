# Proposal

## Why

The pod detail panel (`openspec/changes/pod-detail-panel`) currently renders every field section -
identity, Containers, Volumes, Labels, Annotations, Conditions, Tolerations, Managed Fields - as
one long vertically-scrolling list. That was the right shape to get the content landed at all
(and fixed real bugs along the way: overflow instead of wrap, a shared toolbar toggle instead of
an in-panel one), but a single long scroll makes it hard to jump straight to, say, Containers
without scrolling past everything above it - exactly the organizational problem tabs solve, and
exactly what FreeLens/Lens do for the same view.

## What Changes

- Reorganize the structured field view into tabs: at minimum **Overview** (identity fields -
  Created, Name, Namespace, Labels, Annotations, Controlled By, Status, Node, IPs, Service
  Account, QoS, Termination Grace Period), **Containers** (the container cards, including Init
  Containers), and **Conditions** (badges + Tolerations). Exact grouping is a design.md decision,
  not fixed here.
- The YAML view stays a sibling of the tabbed structured view (the existing toggle), not a fourth
  tab - it's a wholesale alternate representation of the whole pod, not a subset of fields.
- Tab switching keeps the panel's existing keybinding/hint-bar convention: each tab reachable by
  keystroke, not mouse-only.
- Revised after using the first cut (design.md, "Which fields go where"): five tabs - Overview,
  Containers, Volumes, Events, Managed Fields. This adds one thing the flat list never had, an
  Events tab listing the events naming the pod, and gives each Managed Fields entry an
  expandable view of the ownership tree it wrote.

## Capabilities

### Modified Capabilities
- `pod-detail`: reorganizes the structured field view into tabs. No `pod_fields` field is added or
  removed; Volumes and Managed Fields change presentation (always-shown list, per-manager
  expandable blocks), and the Events tab is new content fetched alongside the pod.

## Impact

- `app/src/k8s/resource/pod_detail.rs`: `render_structured`/`render_field` restructured around a
  tab selection; `pod_fields`'s flat `Vec<PodField>` likely needs a grouping key (which tab a
  field belongs to) rather than a full rewrite of the projection itself. `fetch_pod` also lists
  the pod's events.
