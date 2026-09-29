# Proposal

## Why

A pod's log stream can run to thousands of lines with no way to find a specific one short of
scrolling by eye - every comparable tool (k9s, Lens, `kubectl logs | grep`) gives you a filter.
This also has no established pattern in this codebase yet: there is no text-input widget used
anywhere in the app today (checked - `component::input`/`TextInput` has zero call sites), so this
change introduces that pattern for the first time rather than reusing one.

## What Changes

- A filter input in the Logs panel's control bar (alongside the container picker and follow/jump
  controls already landed), narrowing the visible lines to those containing the typed substring,
  case-insensitive.
- The virtualized line list (`uniform_list`) renders the filtered subset - filtering does not
  defeat the earlier virtualization work, it runs before the list sees the line count.
- Matching is substring-only for the first cut; regex/highlight-matches are explicitly out of
  scope here (see design.md for why).

## Capabilities

### Modified Capabilities
- `pod-logs`: adds the requirement that the log panel can be filtered by a substring.

## Impact

- `app/src/util/logs.rs`: `LogsPanel` gains filter state and a text input; `render`'s line-list
  construction filters `view.lines()` before computing `line_count`.
- First use of a text-input component in this app - worth a short look at what gpui-component
  offers (`gpui_component::input`) before committing to a specific widget, since there's no
  existing call site to match conventions against.
