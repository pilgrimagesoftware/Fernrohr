# Proposal

## Why

Opening a row's detail always moves focus to the new panel. Someone working down a list, for
example opening the detail of several failing pods to compare later, has to return to the list
after every open, reselect their place, and continue. Browsers and editors solve this with
"open in background": a modifier-click or modified key adds the tab without leaving where you are.

## What Changes

- Add **Open in Background**: open the selected row's detail panel as a new, inactive tab without
  moving keyboard focus or the list's selection.
  - Mouse: the platform modifier and a click on a row (`cmd`-click on macOS, `ctrl`-click
    elsewhere), or a middle-click.
  - Keyboard: a registered command, by default `cmd-enter` (`ctrl-enter` elsewhere), in every list
    panel's context, guarded with `!Input`.
- If that object's panel is already open, nothing moves. Its tab is not activated and focus stays
  in the list.
- The same gesture on a resource link in a detail view opens the referenced object in the
  background, with the same rules.
- A background panel goes where a normal open would place it. It never becomes the active tab of
  its group, so the list stays visible even when the new tab joins the list's own group.

## Capabilities

### New Capabilities

(none)

### Modified Capabilities

- `resource-browser`: list rows can be opened in the background by modifier-click, middle-click,
  or a keyboard command.
- `resource-links`: links can be followed in the background with the same gestures.
- `app-shell`: "An opened panel takes focus" gains an explicit exception for background opens.

## Impact

- `App/app/src/util/shell/open.rs`: a `background` flag on the single open path
  (`open_target_in`): add the panel without activating its tab or focusing it, and skip the
  focus/activate step for an already-open panel.
- `App/app/src/k8s/resource/pods/` and `object_list/`: row click handlers read the modifier and
  middle button. Add an `OpenInBackground` command per list context.
- `App/app/src/k8s/resource/object_detail/link.rs` (or wherever links dispatch): honor the modifier
  and middle-click.
- Hint rows and the command palette list the new command.
