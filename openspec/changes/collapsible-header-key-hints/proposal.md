# Proposal

## Why

A detail panel's header puts the title first and lets its key hints wrap onto further rows
(App#282, `panel_title::header_title` / `header_hints`). In a narrow panel those hints can wrap
onto four or five rows, so the header grows taller than the content it introduces. The user's
request:

> "regarding the header title / key hints -- if possible, when space is tight and the key hints
> area is going to end up wrapping so much (beyond 3 lines?) that it causes the header to expand
> vertically, try to collapse the key hint area so it shows only 'essential' key hints and hides
> the rest in a tooltip that shows up on hover over the key hint area. That will require
> determining which keybinds are 'essential' and which are not, but it would help when space is
> tight."

## What Changes

- A hint row that would wrap onto more than three lines collapses. It then shows only its
  essential hints and an overflow control in their place. The limit is a named constant,
  `MAX_HINT_LINES = 3`, applied to the row's measured layout rather than to a count of hints. When
  the panel widens enough for every hint to fit in three lines, the row expands again.
- The overflow control is an icon button with a tooltip, following the icon-buttons rule. Hovering
  it shows the hidden hints with their live keys. It is a tab stop, and Enter or Space on it opens
  the window's existing Key Hints overlay (`?`). The overlay lists every command available where
  focus is, hidden hints included. A keyboard user loses nothing: every hidden hint stays in the
  `?` overlay and the command palette, and its key still works.
- Each hint row marks its own essential hints. The detail headers mark the view switches and the
  view toggle. The lists mark Open, Filter and Describe. A row that marks nothing essential keeps
  its first hint.
- The collapse applies to every panel hint row that wraps:
  - the pod detail and object detail headers;
  - the Pods, object list and events browser hint rows.

  These now share one collapsible hint row component. The object list's row doesn't wrap today,
  and starts wrapping as part of this.
- A hint hidden by the collapse is still part of its panel's hint row for every spec that says a
  command "appears in the hint row": it is in the overflow list and the `?` overlay.
- Not covered:
  - Dialogs, the Logs panel's control bar and the cluster picker's key row, which show one key
    beside each control rather than a wrapping row.
  - The shell panel's hint row, which `exec-terminal` requires to always list its reserved keys.

## Capabilities

### New Capabilities

- `key-hint-rows`: how a panel's key-hint row lays out when space is tight. It covers the collapse
  threshold, which hints stay, the overflow control and its keyboard route, and which rows it
  applies to.

### Modified Capabilities

(none. The rows that already "list" or "show" commands keep doing so: a collapsed row's hidden
hints remain listed through its overflow control, as `key-hint-rows` states.)

## Impact

App-only (`App` submodule), no new dependencies:

- `app/src/ui/panel/title/header.rs` (or a new sibling `ui/hint_row.rs`): one collapsible hint row
  component. It measures its wrapped line count, holds collapsed/expanded state with hysteresis,
  and draws the overflow control.
- Each hint row's builder: the pod detail and object detail headers, `pods/hints.rs`, the object
  list's and the events browser's `render.rs`. They mark their essential hints.
- `util/key_hints.rs`: Enter or Space on the overflow control opens its overlay.
- Docs: `docs/site/latest/details.md` and `docs/site/latest/resources.md`, where the hint rows are
  described, plus `docs/site/latest/reference/keybindings.md`'s note on `?`.
