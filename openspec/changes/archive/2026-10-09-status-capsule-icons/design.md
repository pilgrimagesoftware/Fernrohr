# Design

## Context

`ui/status_bar/capsule.rs` renders a `StatusItem` as a ghost button containing the state icon, the
context name, the tunnel name (already `text_xs` and `muted_foreground`), the state text, and
`(<age> ago)`, all tinted by severity. The status bar is frame text, set in Adamina. See
[proposal.md](proposal.md).

## Goals / Non-Goals

**Goals:**

- Make the context name the most prominent text in a capsule and shorten the row.
- Keep every state identifiable without color, now by icon shape alone.

**Non-Goals:**

- Changing severity colors, sorting, the capsule shape, or the menu.
- Changing icons for states that already have a distinct one, beyond what distinctness requires.

## Decisions

### D1. Order and styling

The capsule body renders the context name (Adamina, severity tint), then `[tunnel]` when bound,
then the state icon (severity tint), then the elapsed time when not connected. The tunnel and
elapsed time render in Manrope, one text step below the context name, in `muted_foreground`. The
brackets are part of the muted text, not separate glyphs, so they never take the severity tint.

The state icon moves from the front of the capsule to after the names. The context name then
starts every capsule at the same edge, which is what makes a row of them scannable.

### D2. Tooltip on the state icon

The icon gets its own tooltip, using the app's icon tooltip helper, with the state text. For a
non-connected state, it also gives the elapsed time ("Reconnecting for 42s"), and for a failed or
paused state, the reason. The capsule body's click target is unchanged: clicking anywhere on it,
including the icon, activates the context. Hover only shows the tooltip.

### D3. Distinct icons

Readable-without-color now rests on the icon alone, so every `ContextHealth` state must map to a
different icon. Today's table already does that for connected, waiting for tunnel, reconnecting,
refreshing credentials, and failed. A unit test asserts the mapping is injective, so a future state
(such as `manual-confirmation-tunnels`' awaiting confirmation) cannot reuse one.

### D4. Elapsed time stays visible

The requested layout ends at the icon. Elapsed time for non-connected states stays inline anyway,
in the same muted style as the tunnel name, because the `app-shell` spec relies on it counting up
visibly during a wait or reconnect. Connected capsules, the common case, show exactly
`<context> [<tunnel>] <icon>`.

## Risks / Trade-offs

- [Icons are less self-explanatory than words for new users] -> The tooltip names the state, and
  the icons were already paired with the text, so they are not new.
- [A smaller tunnel name in another font is harder to read at small text sizes] -> It follows the
  configurable text size, one step below the context name, never a fixed pixel size.
