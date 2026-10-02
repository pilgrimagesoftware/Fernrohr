# Proposal

## Why

Closing panels does the wrong thing. With two stacked tab groups, Pods focused in the upper and
RoleBindings in the lower, clicking the lower group's close (x) closes Pods: the group toolbar's
close acts on the focused panel, not on the group it sits in. The last open panel also refuses to
close at all, so the window can never return to the cluster picker by closing panels.
`per-tab-close-button` settled for `Cmd-W` because gpui-component 0.6.6 drew one toolbar per tab
group; gpui-kit 0.7.0, which the app now pins, ships a per-tab close button (hidden by default),
so the original ask is now reachable.

## What Changes

- Every panel tab shows its own close control, closing exactly that tab's panel.
- Any remaining group-level close acts on that group's active panel, never on whichever panel has
  focus elsewhere.
- The last open panel closes like any other; closing it returns the window to the cluster picker.
- Closing a panel with the mouse hands focus on as `Cmd-W` does (`app-shell`'s active-tab focus
  requirement).

## Capabilities

### New Capabilities

(none)

### Modified Capabilities

- `app-shell`: adds per-tab close and correct close targeting, including the last panel.

## Impact

- App: dock/tab setup where the `DockArea`/`TabPanel` is built (enable 0.7's per-tab close), the
  close-action routing, and the window's empty-dock -> picker transition.
- Supersedes `per-tab-close-button`'s deferred Option A; that change keeps its `Cmd-W` work.
