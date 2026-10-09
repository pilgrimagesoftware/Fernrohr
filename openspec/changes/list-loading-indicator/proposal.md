# Proposal

## Why

A list panel's table is empty while its first list request is in flight. On a large cluster, the
Pods panel can sit blank for several seconds, which looks exactly like a namespace with no Pods,
so the user cannot tell whether to wait or whether something is wrong.

## What Changes

- While a list panel is loading its first list, its table shows a loading indicator naming what
  is loading ("Loading Pods…") instead of an empty table, and once rows start arriving it shows
  how many have arrived so far.
- After loading finishes, an empty result says so ("No Pods in team-a"), distinct from loading.
- When a panel relists with rows already on screen (a namespace scope change, or a watch restart
  after a reconnect), the rows stay visible and a small indicator in the panel's header shows the
  refresh, instead of the table blanking.
- The indicator appears only after a short delay, so fast loads never flicker.
- Applies to the generic resource list panels, the Pods panel, and the events browser.

## Capabilities

### New Capabilities

- None.

### Modified Capabilities

- `resource-browser`: list panels show loading, empty, and refreshing states.
- `events-browser`: the events browser shows the same states.

## Impact

- The object list, Pods, and events browser stores (tracking `Init`, `InitApply`, and `InitDone`)
  and their table and header rendering.
- A spinner from gpui-kit, or a small in-app one if the kit has none.
