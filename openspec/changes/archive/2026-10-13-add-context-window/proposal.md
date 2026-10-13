# Proposal

## Why

Adding a context to a window that already has one opens an in-window dialog that blocks the whole
window until it closes, even while a connection or tunnel takes its time (Fernrohr-App#232). The
dialog is also narrower than the picker it hosts, so long generated context names (`gke_…`,
`arn:aws:eks:…`) are cut off and the row's tunnel selector is pushed out of reach.

## What Changes

- In a window that already has a context, Add Context opens a separate, non-modal Add Context
  window. The workspace window stays usable while it is open. Choosing Add Context again raises
  the one already open for that window.
- The Add Context window adds the chosen context to the window it was opened from once the context
  connects, then closes itself. A failed connection stays in the Add Context window.
- A window with no context still shows the full-window picker, as now.
- The picker's card grows with its window instead of sitting at a fixed 480px, so long context
  names have room, and each row's tunnel selector stays inside the card at every width.

## Capabilities

### New Capabilities

None.

### Modified Capabilities

- `app-shell`: Add a context to a window - opens a separate non-modal window rather than a
  blocking dialog.
- `cluster-picker`: Context list from kubeconfig - long names and the tunnel selector stay
  reachable.

## Impact

- App code: `ui/status_bar/capsule.rs` (the add control), a new Add Context window module, the
  picker's card layout (`ui/picker/layout.rs`, `ui/picker/render.rs`).
- Docs: `docs/site/latest/clusters.md` ("Add another context to a window") describes the new
  window.
