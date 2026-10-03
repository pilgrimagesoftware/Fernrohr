# Proposal

## Why

Each window now reports its cluster contexts twice: as chips in the context bar along the top and as
items in the status bar along the bottom, with the same health shown in both. The top bar is also
still the OS-native title area plus a separate context bar, not the gpui-kit Toolbar the rest of the
chrome is moving to. And although a theme preference exists (System, Light, Dark), the app offers no
way to change it.

## What Changes

- **Toolbar**: the window's top bar becomes a gpui-kit Toolbar holding only the app icon and name,
  for now.
- **Capsules move to the status bar**: the context chips (context name, tunnel name, health) move
  into the status bar, merged with its per-context status items so each context appears once,
  carrying its state text and elapsed time. The add-context control and each capsule's disconnect
  action move with them. **BREAKING** (UI): the separate context bar is removed.
- **Theme switcher**: the opposite end of the status bar gets a theme switcher (System, Light,
  Dark) that applies at once and persists the existing theme preference.

## Capabilities

### New Capabilities

(none)

### Modified Capabilities

- `app-shell`: the status bar absorbs the context bar (capsules, add, disconnect) and gains a theme
  switcher; the top bar becomes a gpui-kit Toolbar with icon and name; the context bar requirement
  is removed.

## Impact

- App: window chrome (title bar / context bar / status bar rendering), the add-context popover's
  anchor, the disconnect confirmation's trigger, the theme preference writer in `config::ui`.
- Specs: `typography` still names "the context bar" among Adamina surfaces; that becomes the
  toolbar and status bar when this archives.
