# Proposal

## Why

Labels and annotations render as one chip per entry, each showing its full `key=value`. Tooling
annotations often carry large values: multi-line JSON for monitoring autodiscovery, or a sidecar
injector's single-line status blob hundreds of characters long. One such chip can be taller than
the rest of the field list, or run off the panel's edge, burying the short entries around it. The
full text is rarely what the user needs at a glance, and the YAML view already has it.

## What Changes

- A label or annotation chip whose value is longer than 100 characters or spans more than one line
  shows `key=` followed by the first 20 characters of the value's first line and an ellipsis.
- Hovering a shortened chip shows the full value in a tooltip. The YAML view is unchanged and still
  shows every value in full. Chips are not expandable in place.
- Short values, and every other chip (capacities and the like), render exactly as today.
- This applies in both the pod detail panel and the generic object viewer, which share the chip
  rendering.

## Capabilities

### New Capabilities

(none)

### Modified Capabilities

- `pod-detail`: labels and annotations chips shorten large values.
- `object-detail`: the same rule for the object viewer's labels and annotations.

## Impact

- `App/app/src/ui/detail.rs`'s `chips` helper (shared by both panels) and its callers that build
  label/annotation chips.
- No change to data fetching, the YAML view, or any persisted state.
