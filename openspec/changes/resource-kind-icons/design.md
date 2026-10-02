# Design

## Context

Kinds appear as text in panel title bars, tabs, the Resource panel's kind list, cards and links.
GPUI's `svg()` element draws an SVG as a single-colour mask tinted by the text colour, which suits
monochrome UI icons but would flatten the Kubernetes set's two-colour artwork. The set
(`kubernetes/community/icons`) ships per-kind SVGs in labeled and unlabeled variants, under a
choice of Apache-2.0 or CC-BY-4.0.

## Goals / Non-Goals

**Goals:**
- Full-colour, recognizable kind icons everywhere a kind is named.
- One lookup from kind to icon, with total coverage via fallbacks.

**Non-Goals:**
- No per-CRD icons from CRD metadata or vendor artwork.
- No icons in table cells per row (too dense); kinds only, where they're named.
- No themable or user-replaceable icon packs.

## Decisions

- **Unlabeled variants, rendered in colour.** The labeled variants bake the kind's abbreviation
  into the artwork, which duplicates the adjacent text and doesn't scale to small sizes. Colour
  rendering goes through GPUI's image path (`img()` with SVG source, rasterized at the display's
  scale factor and cached per size) instead of `svg()`, since `svg()` is a mask. A short spike
  confirms `img()` renders the set's SVGs faithfully at 16-24px on 1x and 2x displays before the
  rest is built.
- **Apache-2.0 option.** Its obligations (ship the licence text and attribution) are simple to
  meet in a desktop bundle; the About credit covers attribution either way.
- **Fallbacks drawn to match the set's style** (blue heptagon, white glyph): container (a box),
  custom resource (the set's CRD shape with a generic glyph), generic kind. They're small SVGs
  committed alongside the set, under the project's licence.
- **One lookup**, `ui::icon::for_kind(group, kind) -> IconRef`, keyed by group and kind so a CRD
  named `Pod` in another group doesn't get the Pod icon. A kind the set
  doesn't cover gets the generic kind icon if its group is one the API server itself serves (an
  explicit list), and the custom-resource icon otherwise - not a `*.k8s.io` suffix rule, which would
  misfile Gateway API and volume-snapshot CRDs as built-in.

## Risks / Trade-offs

- [Full-colour icons compete with the one-accent rule in `visual-language`] → Icons are
  identification, not interaction; they're kept small and never used to show state or selection.
  The manual check reviews this on a busy window.
- [Rasterizing SVGs costs memory per size] → Cache per (icon, pixel size); the set is small.
