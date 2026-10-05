# Design

## Context

`k9s-remaining-keybindings` added `K8sPortForward` behind `ForwardRegistry` and an app-wide
`PortForwards` list (`k8s/cluster/port_forwards.rs`). Manage Tunnels reads that list for its Port
forwards section, with a Stop button. The Pods list and the Services list start forwards with
`shift-f`, and report results through a panel-level notice.

## Goals / Non-Goals

**Goals:**

- One observable source of truth for "which forwards does this object have", read by every
  surface.

**Non-Goals:**

- Forwards for kinds other than Pods and Services.
- Remembering forwards across restarts.

## Decisions

### 1. Indexed lookup on `PortForwards`

`PortForwards` becomes an observable GPUI entity, or gets one wrapped around it, that keeps an
index keyed by `(context, namespace, kind, name)`. It exposes:

- `for_object(key) -> Vec<ForwardSummary { id, local_addr, target_port, container }>`
- `stop(id)`, which reuses the existing release path

Surfaces subscribe with `cx.observe`, so starting or stopping a forward anywhere, Manage Tunnels
included, re-renders the rows and panels that show it.

### 2. Surfaces

- **Forwards column:** a narrow fixed-width column added to the Pods and Services column sets. It
  is sortable by count, and its tooltip is built from `for_object`.
- **Detail strip:** a conditional row above the tab bar, following the panel's spacing tokens,
  with no border or focus ring.
- **Container ports:** each port chip gets either a start icon, or the address with copy and stop
  icons.

Icons come from the existing icon set. Every icon button has a tooltip and an id for tests.

### 3. Commands

`PortForwardPod` (`shift-f`) is registered in `PodDetailPanel && !Input` too, sharing
#143's detail-panel action wiring. A new `StopPortForward` is registered in the Pods
list, the Services list and the pod detail contexts, with a default key chosen to pass the
prefix-aware conflict check. The candidate is `ctrl-shift-f`. With several forwards it opens the
same picker style as the port prompt. A container port's start button calls the start path with
that port, which skips the prompt.

### 4. Notifications

Remove the list-panel success notice. Failures use gpui-kit's notification (toast) API,
`window.push_notification`, which the app already uses for other transient errors.

## Risks / Trade-offs

- [Another column crowds narrow lists] → it's narrow, empty for most rows, and can be hidden or
  reordered like any other column.
- [Overlap with #143's detail-panel commands, which are in flight] → implement after #143 merges,
  and reuse its detail-panel action wiring.
