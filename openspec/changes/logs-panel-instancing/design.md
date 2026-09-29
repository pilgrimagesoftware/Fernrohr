# Design

## Two actions, not a modifier read inside one handler

The pod-detail work already settled this question once: `d`/`y` are two distinct actions
(`ShowPodDetail`/`ShowPodDetailYaml`), not one action with an inspected argument, because GPUI's
keystroke -> action dispatch is the idiomatic path and a single action would leave the two
keystrokes indistinguishable at the point that matters. This follows the same shape:
`ShowPodLogs` keeps meaning "open logs, per the configured default"; a new `ShowPodLogsNewPanel`
(bound to `shift-l` in `PodsPanel`'s key context) means "open a new per-pod panel regardless of
the default." For a mouse entry point (a future context-menu item, say), `ClickEvent::modifiers`
is real and available there - that path reads `event.modifiers.shift` directly rather than
needing a second action, since a click handler already has the event in hand.

## The preference decides the *default* action's behavior, not which action exists

`UiConfig.logs_panel_reuse: bool` (default `true`, matching today's behavior) is read at the one
place `ShowPodLogs`'s handler decides which `NavTarget` to open:

```
match (config.logs_panel_reuse, forced_new) {
    (true, false) => NavTarget::Logs,             // reuse (today's behavior)
    (false, false) => NavTarget::PodLogs(pod_ref), // per-pod is the default
    (_, true)      => NavTarget::PodLogs(pod_ref), // shift-l always forces new
}
```

`ShowPodLogsNewPanel`'s handler always resolves to `NavTarget::PodLogs(pod_ref)` - there is no
"reuse, forced" case, since reuse is the one behavior that's already the unmodified default.

## `LogsPanel` needs two shapes, not two panel types

`PodDetailPanel` is constructed once per pod and never reassigned - that's why it doesn't need to
observe `SelectedPod`. A `NavTarget::PodLogs(PodRef)`-backed `LogsPanel` needs the same
non-reactive construction (built from a fixed `PodRef`, streams that pod's logs, done) while the
existing `NavTarget::Logs`-backed one keeps observing `SelectedPod` and re-targeting reactively.
Rather than fork `LogsPanel` into two structs duplicating the stream/follow/container-picker
logic, `LogsPanel::new` takes an `Option<PodRef>`: `Some` pins it (no `SelectedPod` observer
registered at all), `None` is today's reactive construction. This is a smaller, more honest change
than a second panel type that would immediately need to re-implement everything `LogsPanel`
already does.

## Open question for whoever implements this

Whether the preference belongs in `UiConfig` (global, like theme) or should be workspace/per-
window - a user who wants reuse in one workspace and per-pod instancing in another is a real
enough use case to weigh before committing to `UiConfig`'s global scope. Flagging rather than
deciding, since this repo's existing config layering (`ui.toml` vs `workspace.toml`) already
makes that distinction elsewhere and either answer is defensible.
