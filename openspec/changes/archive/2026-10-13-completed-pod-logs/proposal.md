# Proposal

## Why

Opening the logs of a Job's pod after it has completed shows only "Log stream ended." (pilgrimagesoftware/Fernrohr-App#227) - the
output the container wrote is gone from the screen. That is the one moment a person most wants
the logs: a Job finished or failed and they want to know what it said. The `pod-logs` spec already
promises the panel "retains the lines already shown" when a container exits; the app does not
keep that promise for a stream that ends on its own.

## What Changes

- When a pod's stream ends normally, the panel keeps rendering the lines it received and shows the
  "ended" notice after them, instead of replacing them.
- For a container that is already terminated, the panel requests its log without follow, so the
  full output is read to its end rather than relying on a follow stream closing cleanly.
- The "ended" notice is a marker below the content (and an empty terminated container says it
  wrote nothing), not a full-panel error view.
- Previous-instance logs (`previous=true`, already reachable through the existing Previous toggle)
  are unchanged and out of scope here (see design.md).

## Capabilities

### Modified Capabilities
- `pod-logs`: the stream-a-pod's-logs requirement now covers completed pods - their output stays
  visible with an ended marker after it - and follow mode does not apply to a container that has
  terminated.

## Impact

- `app/src/util/logs/render.rs`: `LogsPanel::render` stops swapping the line list for the error
  view when the terminal state is a plain end; the notice renders after the lines.
- `app/src/util/logs/view.rs`: `LogsView` distinguishes an end (a marker alongside lines) from a
  failure (the error view).
- `app/src/util/logs/stream.rs`: `log_params` sets `follow` from whether the container has
  terminated, and `LogTarget` carries that; `app/src/util/logs/panel.rs`'s `restart` fills it in
  from the pod's container statuses.
