# Design

## Context

`namespace-sets` already establishes the pattern this change reuses twice over: a small
preference file under `preference_dir()` loaded/saved through the existing `config::load`/`save`
pair, and a config-mutation path that notifies every open window immediately (its store already
has to do this for "a pin made in one window is visible in another" - `namespace-sets`' own sets
are global for the same reason). `object-detail`'s `ObjectRef` (kind, namespace, name, cluster
context) is already the identifier used for every reference and link; a pin is a `Vec<ObjectRef>`
with no new identifier concept.

See proposal.md for why; see the three spec deltas for exact requirements.

## Goals / Non-Goals

**Goals:**
- Pinning is the same one identifier (`ObjectRef`) used everywhere else an object is referenced -
  no second "favorite-able thing" type.
- A pin is visible and actionable from everywhere that object already appears: its own panel, and
  any link pointing to it.

**Non-Goals:**
- Favorite *kinds* or *searches* (e.g. "always show me Deployments in team-a") - this change pins
  individual objects only, matching the request.
- Reordering or organizing pins into folders/tags - a flat, cluster-grouped list is enough for the
  first slice; nothing here blocks adding that later.
- A picker inside the Favorites panel to add a pin by searching - avoided deliberately (see
  proposal.md) since the Resource panel and resource-browser filter already are that picker;
  building a second one duplicates it for no new capability.

## Decisions

**Pins are a flat `Vec<ObjectRef>` in their own preference file, not per-cluster files.** One
`favorites.toml` under `preference_dir()`, `FavoritesConfig { pins: Vec<ObjectRef> }` -
`ObjectRef` already carries the cluster context, so grouping by context for display is a sort/
group-by at render time, not a storage-level split. This mirrors `namespace-sets`' "one file,
not per-cluster" decision and for the same reason: pins are meant to follow the user across
clusters and windows.

**Pin/unpin is one command pair, dispatched with the target `ObjectRef` as its argument, not a
registered command per object.** Same reasoning `namespace-sets` decision 3 already used for
quick-selection: the command registry's `Command` takes a fixed action, so `PinObject`/
`UnpinObject` are two commands whose action carries whichever `ObjectRef` the pin control they're
attached to (the panel's toolbar button, or a link's pin toggle) was built for - not N commands
for N pinned objects.

**A config update notifies open windows the same way `namespace-sets` already has to.** Whatever
mechanism that change lands for "a set created in one window appears in another's quick-selection
picker immediately" (an `Entity` the config loads into and every window observes, per its own
design) is the same mechanism favorites' pin list uses - this change does not invent a second
cross-window notification path.

**The Favorites panel is one panel kind, not one per cluster.** It lists every pin across every
context the running application knows about (not just contexts the current window uses), grouped
by context, consistent with pins not being window-scoped. Activating an entry for a context the
window doesn't currently use follows the same "open or focus" convention `resource-links` already
defines, reusing whatever that link-follow path does when the target context needs connecting
first (if anything beyond "the context must already be connected somewhere" - scoped as an open
question below if it turns out to matter).

**A deleted object's pin is not auto-removed.** Per the spec, a stale pin still lists and, when
activated, shows the existing "object doesn't exist" panel state (`object-detail`'s own
requirement) - no new "pin cleanup" sweep is needed; the existing not-found panel state already
covers it.

## Risks / Trade-offs

[A long-lived pin list accumulates stale entries for deleted objects] → Accepted for this slice:
unpinning is one keystroke/click from the Favorites panel once a stale entry is noticed; an
automatic prune is a plausible follow-up, not required by the request.

[Cross-window pin sync depends on `namespace-sets`' notification mechanism existing first] →
Sequencing risk, not a design risk: if `namespace-sets` lands a different mechanism than assumed,
this change's task 1.2 adopts whatever that mechanism turns out to be rather than building a
parallel one.

## Open Questions

- Does activating a Favorites entry for a cluster context the window doesn't currently use need
  to open/attach that context first, or is it only reachable when the context is already
  connected somewhere? Resolve during task 3.2 by checking what `resource-links`' own
  cross-context follow currently assumes - this does not change the pin/unpin requirements
  themselves, only how far the "open or focus" convention reaches.
