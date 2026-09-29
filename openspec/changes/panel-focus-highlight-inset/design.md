# Design

## Inset, not corner-radius matching

Two ways to stop the border colliding with the OS's rounded corner: round the border itself to
match the window's corner radius, or pull the border in a few pixels so it never reaches the
window edge in the first place. Rounding requires knowing the OS's exact corner radius (an
undocumented, version-dependent value) and only fixes the corner case specifically. Insetting
(`.p_px()`/a small fixed margin on `focus_frame`'s outer `div`, applied to the content it wraps
rather than the border's own edge) fixes it regardless of corner radius, and also reads better
generally - a border flush with a window edge looks cramped independent of the rounding issue.

`focus_frame` keeps `.size_full()` on its outer element (so it still fills the space the dock
gives it) but wraps `content` in an inset inner element that the border is drawn against, rather
than drawing the border on the full-size outer element directly.
