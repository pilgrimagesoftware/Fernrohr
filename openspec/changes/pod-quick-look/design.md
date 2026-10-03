# Design

## Context

See proposal.md. The Pods table rows come from the shared Pod watch, so the popover's core fields
are already in memory; pod detail has the field projection and (from
`resource-detail-ui-improvements`) status colors; `pod-events-time-window` has a per-pod event watch.

## Goals / Non-Goals

**Goals:** a fast, keyboard-driven popover over the Pods table; a row context menu.

**Non-Goals:** quick look for other kinds (the same popover can later take an object-detail
projection); editing anything from the popover.

## Decisions

### D1: Popover built from the table's own Pod, live
The popover reads the selected `Pod` from the shared Pods store and re-renders on its updates, so it
costs no extra API call and stays live. Fields reuse pod detail's projection helpers rather than a
second copy.

### D2: Latest warning from a per-pod event watch, started lazily
Only the most recent Warning needs events, so the popover starts `pod-events-time-window`'s per-pod
watch when it opens (or retargets it on Up/Down) and drops it on close. The rest of the popover shows
at once; the warning line fills in when the watch's initial list arrives.

### D3: Focus stays on the table
The popover is anchored to the selected row and does not take focus: Up/Down keep moving the table
selection, and the popover's own keys (Space, Escape, Enter) are bound in the Pods panel context
while it is open. Open Details is still a focusable button for mouse and Tab users.

### D4: Context menu via gpui-kit's context menu on the row
Right-click selects the row, then shows the menu built from the registered commands (Quick Look,
Open Details, Logs, YAML) so labels and keys stay in sync with the palette.

## Risks / Trade-offs

- [Space conflicts with a future multi-select] -> none exists today; revisit if selection grows.
- [Rapid Up/Down churns event watches] -> debounce the watch retarget (about 250 ms); the
  popover's pod fields update immediately.
