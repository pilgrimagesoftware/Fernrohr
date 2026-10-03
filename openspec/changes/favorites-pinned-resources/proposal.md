# Proposal

## Why

The resource panel's kind tree and resource-browser lists are good for browsing, but a user who
checks the same handful of objects repeatedly - a specific Deployment, a troublesome Pod, a
particular Node - has no shortcut to them: every visit means navigating the kind tree and
filtering again. Named namespace sets (`namespace-sets`) solve this for a namespace scope; nothing
solves it for individual objects.

## What Changes

- **Pin an object**: any object with a detail panel (per `object-detail`/`pod-detail`) gains a
  pin/unpin action, available from its panel and from `resource-links` wherever it is referenced.
- **Favorites list**: a new panel, opened like any other dockable panel, listing every pinned
  object across all clusters the window uses, grouped by cluster context, each entry showing the
  object's kind icon and name and opening or focusing that object's detail panel when activated.
- Pinned objects persist across restarts and are not scoped to one window - pinning in one window
  shows the pin everywhere.
- Pinning a specific object does not depend on that object's panel currently being open; the
  Favorites panel itself offers no separate "add" flow beyond following a pin made elsewhere, to
  avoid building a second object picker that duplicates the Resource panel's.

## Capabilities

### New Capabilities

- `favorites`: pinning an object, persisting pins, and the panel that lists them.

### Modified Capabilities

- `object-detail`: object panels gain a pin/unpin action.
- `resource-links`: a reference can show pinned state and offer pin/unpin without opening the
  target's panel first.

## Impact

- `App/src/config/favorites.rs` (new): `FavoritesConfig { pins: Vec<ObjectRef> }`, persisted
  through the existing `config::load`/`config::save` pair, following the same
  `preference_dir()`-file treatment as `namespace-sets.toml`.
- `App/src/ui/favorites/` (new): the Favorites panel and its list rendering, reusing
  `resource-icons` for kind icons and the existing panel-opening path for activating an entry.
- `App/src/k8s/resource/object_detail/`: pin/unpin action on the object panel's toolbar.
- `App/src/ui/link.rs` (or wherever `resource-links` renders a reference): an optional pin toggle
  alongside a link, matching the Secret-reveal-control precedent of a small per-row control that
  does not require opening the referenced object.
- `App/src/command.rs`: `PinObject`/`UnpinObject` and `OpenFavorites` registered commands.
- Depends on nothing unreleased; `resource-icons` and `resource-links` are already specified and
  (per their presence in `openspec/specs/`) implemented.
