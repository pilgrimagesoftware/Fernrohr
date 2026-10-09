# Proposal

## Why

A status-bar capsule reads `<context> <tunnel> Connected`, with the context name, tunnel name, and
state text all in the same frame font. On a window with several contexts, the row gets long, and
the context name, which is what the user scans for, does not stand out from the text around it.

## What Changes

- A capsule reads `<context> [<tunnel>] <state icon>`.
- The connection state is shown by its icon alone. The state text (Connected, Waiting for tunnel,
  Reconnecting, Failed, and so on) moves into the icon's tooltip, together with how long the context
  has been in that state.
- A bound tunnel's name is shown in square brackets, in the data font (Manrope), one step smaller
  than the context name, and in the theme's muted foreground color.
- A non-connected capsule still shows its elapsed time after the icon, in the same small, muted
  style as the tunnel name, so a wait or reconnect visibly counts up. A connected capsule shows no
  elapsed time, as today.
- Severity colors, sort order, the capsule shape, and the capsule menu are unchanged.

## Capabilities

### New Capabilities

- None.

### Modified Capabilities

- `app-shell`: the Window status bar and Status severity is visually distinct requirements change
  how a capsule presents its tunnel and state.
- `typography`: the status bar's tunnel names render in the data font, as an exception to frame
  text in the status bar.

## Impact

- `ui/status_bar` capsule rendering and its tests.
- `StatusItem`: state text becomes tooltip content.
- `manual-confirmation-tunnels` (not yet implemented) is updated alongside this proposal so its
  awaiting-confirmation capsule uses an icon with an "Awaiting confirmation" tooltip.
