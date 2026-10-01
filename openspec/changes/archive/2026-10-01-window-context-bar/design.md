# Design

## Context

See proposal.md for motivation. Current state in the App repo:

- `WindowMode::Workspace` holds one `context_name`, the dock, the Resource panel, `open_panels`,
  and a `connection_count` fixed at 1. The panel title bar (`ui/panel/title.rs`) already adds the
  context to a panel title when `connection_count > 1`, and the Resource panel
  (`ui/panel/resource.rs`) already has a cluster dropdown for more than one connection.
- `config/workspace.rs` already stores `cluster_context` on **each saved panel**, so a window's panels
  can already name different contexts on disk.
- `ClusterRegistry` (`k8s/cluster/session.rs`) creates a `ClusterSession` per context on first
  `connection()` and **never removes one**. Watches are refcounted per panel, but the session, its
  `ClusterConnection`, and the tunnel `RegistryHandle` that connection holds live until the app exits.
- Dock layouts are also saved per context (`config/dock_layouts.rs`) for the per-cluster layout
  restore.

## Goals / Non-Goals

**Goals:**
- A window owns an explicit, ordered list of contexts. Panels, the Resource panel, the context bar,
  and the status bar all read that list.
- Sessions and their tunnels actually shut down when no window uses them.

**Non-Goals:**
- Changing how per-panel watches are refcounted. That stays as it is.

## Decisions

### 1. A window's contexts are an explicit list, not derived from its panels

`WindowMode::Workspace { contexts: Vec<String>, active: usize, .. }` replaces `context_name`.
Closing a context's last panel by hand does **not** remove its chip, because Disconnect is the only
way to remove one. The Resource panel still lets you open a kind for a context with no panels.
`connection_count` becomes `contexts.len()`.

*Alternative:* derive the contexts from the open panels. It's rejected because a chip would vanish,
and its connection could drop, as a side effect of closing a tab. That's the silent-disconnect
surprise the confirmation exists to prevent.

`connection-status-bar` already lists "the contexts the window uses", so it reads this list with
no spec change once both have landed.

### 2. Window holds on sessions

`ClusterRegistry::hold(cx, context, window_id)` and `release(cx, context, window_id)` keep a set of
holders per session. Adding a context to a window, or restoring it, takes a hold. Disconnecting
the context, or closing the window, releases the hold. On the last release the registry removes the
session. Dropping it drops the `ClusterConnection`, whose `RegistryHandle` drop releases the tunnel
forward through the existing refcount. Panels must be closed before the release so their watch
unsubscribes run first.

*Alternative:* count panels instead of windows. It's rejected for the same reason as decision 1: a
context with no panels still belongs to the window.

### 3. Persistence

`WindowLayout` gains `contexts: Vec<String>` with a serde default. When it's missing (older files),
it's derived from the distinct `cluster_context`s of the window's saved panels, in first-seen order.
Restore takes a hold on each context before panels are rebuilt.

The save side was missing: `layout_from_bounds` only ever wrote geometry, so relaunch always showed
the picker and a window's saved panels were always empty. Saving now records the live window's
`contexts`, so a relaunch reconnects them directly. The dock arrangement stays in `dock_layouts.json`:
a single-context window keeps its bare context-name key, and a multi-context window uses its
contexts sorted and joined with U+001F, so the key doesn't depend on order. (An earlier draft said a
multi-context arrangement lived in the window's saved panels; it can't, because panels aren't saved.)

### 4. Context bar placement and module

`ui/context_bar.rs` renders between the title bar and the workspace body. `MainWindow`'s workspace
branch becomes a column: context bar, body row (`flex_1`), status bar. A chip click sets `active`,
which drives the Resource panel's cluster dropdown, and the dropdown writes `active` back, so the
two stay in sync. The chip menu holds Disconnect. The "+" control opens a popover reusing the
picker's context list filtered by `contexts`. It's a thin wrapper around the picker view, not a
second implementation, so tunnel labels and connect progress come for free. That includes a failed
connect: the popover keeps showing the picker's own failure, and no chip is added until the context
connects.

### 5. Disconnect confirmation

Counting "panels that will close" uses `open_panels` filtered by context. Counting "other windows
still using it" uses the holder count from decision 2 minus this window. The confirmation uses
gpui-component's dialog. The panel close order is: close the panels, release the hold, then return
to the picker if `contexts` is empty.

## Risks / Trade-offs

- [Session teardown is new. Anything caching an `Entity<ClusterConnection>` past the release keeps
  it alive] → A test asserts the tunnel forward is released, which is observable as a
  `ForwardRegistry` entry disappearing, after the last window disconnects.
- [Removing `context_name` touches every `WindowMode::Workspace` match in the 1889-line `shell.rs`]
  → A mechanical first task that changes no behavior, so later tasks diff cleanly.
- [Order of changes] → `connection-status-bar` is a dependency for the chip's health dot.
  Implement it first, or ship the chips without the dot and add it when the status bar lands.
