# Design

## What reading the code does and doesn't rule out

`ClusterPicker::select` (`app/src/ui/picker.rs`) looks correct in isolation: it calls
`emit_connected` synchronously right after starting the attempt (covering the case where the
looked-up connection is already `Connected`), and separately subscribes via `cx.observe` for the
case where it connects later. `MainWindow::watch_picker` (`app/src/util/shell.rs`) subscribes to
`PickerEvent::Connected` and calls `enter_workspace`. Neither reading turns up an obvious
per-window bug - which is exactly why this is scoped as diagnose-first rather than
diagnose-and-fix-blind.

## Leading hypothesis: the registry returns an already-`Connected` entity, and something about
## the second window's subscribe timing misses both firing paths

`ClusterRegistry::connection` (section 2's rekeyed registry) returns the **same**
`Entity<ClusterConnection>` for a given context name regardless of which window asks - that's the
point of the registry (one live connection per context, shared across windows/panels). If a first
window already connected context `X`, a second window's picker selecting `X` gets back an entity
that is already `ConnectionState::Connected` and will never transition again (no further state
change to `cx.observe`). `select`'s synchronous `self.emit_connected(cx)` call right after
`new_connection` is exactly the code that's supposed to handle this - so if this hypothesis is
right, the bug is specifically in why that synchronous call doesn't take effect for the second
window's case (e.g. an ordering issue between `watch_picker`'s subscribe and `select`'s
synchronous emit, or the second window's picker being constructed via a different path than the
first that skips one of these steps).

## Alternative hypothesis: a genuinely fresh connect's task never reaches the second window

If the second window is connecting to a *different*, not-yet-connected context, the relevant path
is `cx.observe`'s callback firing once the connection later transitions to `Connected` off a
background task. If that task's completion is somehow tied to the *first* window's context (a
stale window/cx captured in a closure, or a global that only ever notifies one subscriber), the
second window would silently never receive the update. This is the scarier hypothesis, since it
would point at a lifetime/capture bug in the connect path itself, not a narrow timing issue.

## How to tell them apart

The reported symptom ("says connected but doesn't switch") matches the *first* hypothesis better
- the picker's own status text presumably reads `ConnectionState` directly (so it correctly shows
"connected"), while the mode-switch depends on the event firing, which is the piece that's
failing. A regression test that opens two windows and connects the second one to a context the
first window has **already** connected is the fastest way to confirm or rule out hypothesis one
before looking anywhere else.
