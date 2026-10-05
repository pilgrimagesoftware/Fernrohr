# Design

## Context

The pod detail panel (`App/app/src/k8s/resource/pod_detail/`) is its own `KeyContext`
(`PodDetailPanel`) with a fixed set of panel-local commands registered in `commands.rs`: view
toggle (`y`), six tab keys (`1`-`6`), hide-secret-values (`h`), YAML fold/unfold, and copy name.
Logs today open only from the Pods list panel (`pods/commands.rs`'s `l` key), which sets the
app-scoped `SelectedPod` global (`pods/selection.rs`) with namespace, name, container names and
context, then dispatches the window-global `ShowLogs` action (`ui/nav.rs`), which the window's
`on_action_show_logs` handler turns into an open/focus of the Logs panel.

The detail panel already knows everything `PodSelection` needs: `self.pod` (namespace, name),
`self.scope.context_name`, and, once loaded, `self.state`'s `Box<Pod>` for `spec.containers`.

## Goals / Non-Goals

**Goals:**
- One keystroke, consistent with the Pods list's own `l`, opens logs for the pod a detail panel
  is already showing.
- Reuse the existing `SelectedPod` + `ShowLogs` path exactly - no second way to open Logs, no
  changes to the Logs panel or `pod-logs` capability.
- Palette-visible and `keymap.toml`-overridable, like every other panel-local command.

**Non-Goals:**
- Jumping to a specific container's logs from the Containers tab's per-card detail. This change
  only reaches logs for the pod as a whole, defaulting to the first container the same way the
  Pods list's `l` does.
- Changing the Logs panel's container-selection UI.

## Decisions

- **New action `ViewLogs`, key `l`, registered in `pod_detail/commands.rs`.** `l` is free in the
  `PodDetailPanel` context and is the same key the Pods list already uses for the same intent, so
  it reads as one shortcut users already know rather than a new one to learn.
- **Handler builds `PodSelection` from panel state, not a fresh fetch.** While `self.state` is
  `PodDetailState::Loaded(pod)`, the handler reads `pod.spec.containers` for names exactly as
  `pods/render.rs` does when building a row's `PodSelection`, sets `SelectedPod`, and dispatches
  `ShowLogs`. While still `Loading` or on fetch failure, the command is a no-op - there is no pod
  to view logs for yet, and no existing command in this panel acts before load completes either.
- **No new nav/global state.** The command's only job is producing the same `PodSelection` the
  Pods list already produces and handing it to the same global + action. This keeps Logs's
  container-selection and streaming behavior (`pod-logs` capability) completely unchanged.

## Risks / Trade-offs

- Minor: a detail panel opened directly (e.g. restored from a saved layout) before its first
  fetch completes will have the command briefly do nothing if pressed immediately. Acceptable -
  the same window applies to the existing tab-switch commands reading `self.state`.
