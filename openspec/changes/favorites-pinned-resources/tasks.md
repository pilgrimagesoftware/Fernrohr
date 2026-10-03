# Tasks

## 1. Pin storage

- [ ] 1.1 Add `config/favorites.rs` with `FavoritesConfig { pins: Vec<ObjectRef> }` and pure
  helpers `pin`, `unpin`, `is_pinned`, `pins_for_context`, loaded/saved through the existing
  `config::load`/`config::save` pair at `preference_dir()/favorites.toml`; verify with unit tests
  covering pin-idempotence, unpin-of-unpinned (no-op), and grouping by context.
- [ ] 1.2 Wire the same cross-window notification mechanism `namespace-sets` uses for its sets
  (an observed `Entity`, or whatever that change lands) so a pin made in one window appears in
  another window's open Favorites panel without reopening it; verify with a two-window test.

## 2. Pin/unpin commands

- [ ] 2.1 Register `PinObject`/`UnpinObject` commands taking a target `ObjectRef`, each toggling
  the pin and showing the opposite label once applied; verify both appear in the command palette
  and accept a `keymap.toml` override by id.
- [ ] 2.2 Add the pin/unpin control to the object detail panel's toolbar, reading pinned state
  from `FavoritesConfig`; verify a panel for a pinned object shows "Unpin" and an unpinned
  object's panel shows "Pin", with a keyboard test exercising the toggle.
- [ ] 2.3 Add the same pin/unpin control to `resource-links` references (a small toggle beside
  the link, matching the Secret-reveal-control precedent of a per-row control that doesn't
  require opening the target); verify pinning through a link does not open the target's panel.

## 3. Favorites panel

- [ ] 3.1 Build the Favorites panel listing pins grouped by cluster context, each entry showing
  its kind icon (`resource-icons`) and name, with an empty-state message when there are no pins;
  verify with a test asserting group boundaries and the empty-state message.
- [ ] 3.2 Activating an entry opens or focuses that object's detail panel following
  `resource-links`' open-or-focus convention, resolving design.md's open question about a
  context the window doesn't currently use; verify with a test for an already-open target
  (focuses) and a not-yet-open target (opens).
- [ ] 3.3 A pinned object that no longer exists still lists, and activating it opens a panel
  showing it doesn't exist, reusing `object-detail`'s existing not-found state; verify with a
  test against a pin for a deleted object.
- [ ] 3.4 Register `OpenFavorites`, keybound and in the command palette; verify the panel opens
  and is keyboard-navigable (`simulate_keystrokes` moving between entries and activating one).

## 4. Integration and documentation

- [ ] 4.1 Run `cargo fmt && cargo clippy -- -D warnings && cargo test` and fix any failures.
- [ ] 4.2 Confirm `App/CLAUDE.md`'s keyboard-first checklist items (palette entry, keymap
  override, visible hint) hold for all three new commands by re-reading the panel and toolbar
  code against that checklist.
