# Design

## Where the indicator lives: the panel's title element

The indicator has to be on the tab (Paul's review), and gpui-component 0.6.6 gives a panel exactly
one way onto its tab: `render_tabs` draws `panel.title()` as the label whenever `tab_name()` is
`None`. Fernrohr's panels already return `None` from `tab_name` so the tab carries the title's
context tooltip. So the focus state goes into `panel_title::title_element`, which every panel's
`title()` already calls. The single-panel title bar (`render_title`) draws the same element, so it
is marked the same way.

What this cannot do is colour the tab's own background: that belongs to the library's `Tab`, and
no per-panel style reaches it. An underline on the label is the strongest mark available without
patching the crate.

## Focus test: `contains_focused`, not `is_focused`

Same reasoning as the removed border: a panel whose content takes focus itself (a table row, a
text input) moves the window's focus to that child. An indicator lit only while the panel's own
handle is focused would go dark the moment the panel was actually used.

## Colour: the user's accent colour

On macOS the underline uses the user's accent colour from System Settings
(`NSColor.controlAccentColor`, converted to sRGB through `objc2-app-kit`'s safe bindings; the crate
was already in the build through gpui's macOS backend). Elsewhere, or if AppKit refuses the
conversion, it falls back to the theme's `blue` (`blue-600` light / `blue-400` dark). Not `primary`:
the removed border used it, but in both default themes `primary` equals the selected tab's
foreground (`neutral-900` / `neutral-50`), so it would mark nothing.

The colour is cached in a global (`ui::accent`), since titles render every frame, and re-read at
startup, on every light/dark change, and when a main window becomes active. Activation is how a
change made in System Settings shows up: the user has to switch away to make it, and the colour is
re-read when they come back. Linux and Windows accent lookups are future work.

## No layout shift

Every title reserves a 2px bottom border; only its colour changes (`transparent_black()` when
unfocused). Moving focus therefore never moves any tab's label.

## Redraw on focus change

`Window::focus` calls `refresh()`, which sets `refreshing`, and the dock's cached panel views are
re-rendered while `refreshing` is set. The tab strip therefore picks up the new focus state on the
same frame.

## Focusable panels

`track_focus` does two things: it puts the handle in the dispatch tree (so `contains_focused` can
see a focused child), and it focuses the handle on mouse-down. Pods and Pod detail already tracked
their handle. Placeholder and Logs did not, so a click inside them focused nothing, and they gained
it here.
