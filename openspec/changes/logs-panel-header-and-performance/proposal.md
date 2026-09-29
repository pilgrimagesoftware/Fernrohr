# Proposal

## Why

The Logs panel's title bar shows only the generic "Logs" label, with no indication of which pod
or container is streaming - `LogsPanel` already tracks `current: Option<(String, String,
String)>` (namespace, pod, container) internally but never surfaces it, so a user with more than
one Logs panel open (or who just switched panels) has to guess. Separately, scrolling is reported
hitchy and slow: `LogsPanel::render` builds one GPUI `div` per log line unconditionally
(`view.lines().iter().cloned().map(|line| div()...)`), with no virtualization - a pod with
thousands of lines means thousands of live elements every render, which is exactly the kind of
workload GPUI's virtualized list primitives exist for and plain `.children()` does not scale to.

## What Changes

- The Logs panel's title bar shows the pod name and container it's streaming, not just "Logs".
- Log lines render through a virtualized list (rendering only the visible line range, not every
  line unconditionally) so scroll performance stops degrading with log volume.

## Capabilities

### Modified Capabilities
- `pod-logs`: adds the requirement that the log panel identifies which pod and container it is
  showing.

## Impact

- `app/src/util/logs.rs`: `LogsPanel::title`/`tab_name` (or wherever its `Panel` impl currently
  returns a bare "Logs" label) read from `self.current`; `LogsPanel::render`'s line-rendering
  block replaces the unconditional `.children(...)` map with a virtualized list.
