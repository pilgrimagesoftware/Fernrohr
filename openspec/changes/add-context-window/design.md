# Design

## Context

The add control (`StatusBarView::open_add_dialog`) builds a `ClusterPicker` that excludes the
window's contexts and puts it in a 480px `window.open_dialog`. A dialog is modal to its window,
and the picker's card is itself a fixed 480px plus padding, so inside the dialog the card's right
edge - where each row's tunnel selector sits - is clipped.

The status bar exists only in `WindowMode::Workspace`, so "a window with an active context" is
exactly the case the add control serves. A window with no context is in `WindowMode::Picker` and
shows the full-window picker, which this change leaves alone.

## Goals / Non-Goals

**Goals:**
- The add flow never blocks the workspace window.
- One Add Context window per workspace window.
- The tunnel selector is reachable for any context name length.

**Non-Goals:**
- Horizontal scrolling of the context list. Ellipsis plus hover tooltip plus a resizable, wider
  card already shows the full name and keeps the selector visible.
- Changing what the picker shows or how connecting works.

## Decisions

1. **A real window, not a non-modal in-window panel.** gpui-kit dialogs are modal by design and
   a popover closes on outside click, which would cancel a slow connect. A separate window, like
   About and Tunnels, is non-modal for free and can be moved aside. It is rooted with
   `gpui_kit::open_window` like every other window, so dialogs and keyboard traversal work inside.

2. **The window hosts the existing `ClusterPicker`.** Tunnel selection, connect progress, manual
   tunnel confirmation and failure reporting all come from the picker already. Its
   `PickerEvent::Connected` drives the add, exactly as the dialog's subscription did.

3. **Adding goes through the workspace window's own handle.** `MainWindow::add_context` takes the
   `Window` it holds the context for (the registry hold is keyed by window id, and the Pods panel
   opens in that window's dock). The Add Context window keeps the opener's `AnyWindowHandle` and
   updates `MainWindow` through it, never with its own `Window`.

4. **Single instance per workspace window, kept by the status bar.** `StatusBarView` stores the
   `AnyWindowHandle` of its Add Context window. Opening again activates it if updating it still
   succeeds - the same closed-window test the About window uses - and otherwise opens a new one.
   If the workspace window closes first, a later Connected finds the opener gone and the Add
   Context window just closes.

5. **Escape is a registered command.** `add_context.close`, bound to Escape in the window's own
   `AddContextWindow` key context, so a deeper context (an open tunnel selector) still takes
   Escape first, and the palette offers it.

6. **The row shrinks, and the card fills its window up to a cap.** The row's missing zero
   minimum width was the clipping bug: a long name never ellipsized, so the selector was pushed
   past the card. The row now has `min_w_0`. The card becomes `w_full` with the old 480px as
   its minimum and a maximum of 800px, inside a column padded by the existing side margin. The
   full-window picker gets the same room for long names. Names still ellipsize with a hover
   tooltip; the selector is `flex_shrink_0`, so it always stays inside the card.

## Risks / Trade-offs

- [The picker's own window can be narrower than a long name] → ellipsis plus tooltip, and the
  window is resizable and opens wider than the 480px card.
- [The excluded list is a snapshot from when the window opened] → `add_context` already ignores a
  context the window already uses, so a stale row cannot add a duplicate.
- [Two windows to keep track of] → the Add Context window brings its opener to the front and
  closes itself on success.
