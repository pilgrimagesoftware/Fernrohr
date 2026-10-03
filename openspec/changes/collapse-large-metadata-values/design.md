# Design

## Context

`ui::detail::chips` renders a list of preformatted `key=value` strings as wrapping chips, and both
the pod detail panel and the object viewer feed it labels, annotations and capacities. Values are
drawn in full, so a large value sets the chip's size. GPUI elements already support `.tooltip(...)`
(used by the context bar and tunnel picker).

## Goals / Non-Goals

**Goals:**
- Keep every label/annotation chip one line and bounded in width.
- Keep the full value one hover away, and in the YAML view, which is also its keyboard route: the
  panel's existing YAML toggle shows every value in full, so no chip needs to be a tab stop.

**Non-Goals:**
- No click-to-expand on chips. A shortened chip does get the shared copy control (from `resource-detail-ui-improvements`) as its keyboard route to the full value: a Tab stop that copies it on Space or click. Short chips have none.
- No change to capacity chips, conditions badges, or the YAML view.
- No configurable threshold.

## Decisions

- **Shorten at the chip, from structured input.** Label/annotation callers pass `(key, value)` pairs
  to a metadata-chip variant rather than preformatted strings, so the split never guesses at an `=`
  inside the key or value. Capacity chips keep the existing string path.
- **Same rule as the pod Configuration tab's ConfigMap values** (large past 100 characters or one
  line, previewed as the first 20 characters of the first line), so large values read the same
  everywhere. The preview counts characters, not bytes, so it never splits a
  UTF-8 sequence.
- **The tooltip is built from the same value the chip was given.** For a Secret, callers already
  pass the redacted placeholder, so the tooltip can't reveal more than the chip.
- Alternative considered: expand-in-place like ConfigMap values. Rejected at the user's request;
  metadata is reference material, and in-place expansion brings back the layout jump this fixes.

## Risks / Trade-offs

- [A tooltip with a very large value could be bigger than the window] → Wrap the tooltip text and
  cap its width; the YAML view remains the place for reading a value in full.
