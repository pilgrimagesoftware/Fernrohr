# Design

## Context

`NamespaceScope` (`App/app/src/config/workspace.rs`) is currently `All | Single(String)`, stored
per-panel in `PanelDescriptor::Pods`. The Pods panel's only mutator is the single-shot `WarpNamespace`
command (`pods/commands.rs`), which replaces the scope outright with `Single(selected_pod.namespace)`.

This change depends on `warp-all-to-namespace` for the per-context default-scope propagation
mechanism (a context -> default `NamespaceScope` map, applied to existing panels and used as the
initial scope for new ones). This design assumes that mechanism exists and reuses it rather than
building a second propagation path.

## Goals / Non-Goals

**Goals:**
- Extend `NamespaceScope` with an include/exclude set variant that composes (add/remove) instead
  of replacing the whole scope.
- Add panel-local and "for all" commands for include/exclude, following the existing command
  registration pattern (`CommandRegistry`, `actions!`, keymap-overridable binding).
- Reuse `warp-all-to-namespace`'s per-context default-scope propagation for the "for all" variants.

**Non-Goals:**
- A dedicated namespace picker UI/dialog for choosing namespaces arbitrarily (not just "the
  selected row's namespace") - out of scope here; commands act on the currently selected row's
  namespace, same as today's `WarpNamespace`.
- Extending namespace scoping to resource kinds beyond Pods - the type change is shared, but wiring
  other panels is future work tracked separately.

## Decisions

**`NamespaceScope` gains `IncludeExclude { mode: IncludeExcludeMode, namespaces: BTreeSet<String> }`**
rather than two separate variants (`Include(Set)` / `Exclude(Set)`). One variant with a mode field
keeps match arms that don't care about the distinction (e.g. "is this scoped at all") simple, and
makes toggling between include and exclude a field flip instead of a variant change.
`BTreeSet` over `Vec` for stable serialization order (predictable `workspace.toml` diffs) and
natural add/remove/contains semantics.

**Collapsing an empty exclude set back to `All`, and an empty include set to... itself (empty
include = show nothing)**: removing the last excluded namespace is common (undoing a mistaken
exclude) and should feel like "back to normal," so it collapses to `All`. An empty include set has
no natural collapse target and is a legitimate, if unusual, "show nothing" state - left as-is
rather than guessing the user's intent.

**Include/exclude act on the selected row's namespace**, matching `WarpNamespace`'s existing
interaction model, rather than opening a multi-select dialog. Keeps this change additive to the
existing command, not a UI redesign. A namespace-set editor dialog can layer on top later without
changing the underlying `NamespaceScope` representation.

**"For all" reuses `warp-all-to-namespace`'s propagation, not a new one.** Both features need the
same thing: apply a scope change to every open panel in a context, and remember it as that
context's default. Building two parallel mechanisms would double the surface for bugs like "panel
updated but default didn't" or vice versa.

## Risks / Trade-offs

[Older `workspace.toml` files predate `IncludeExclude`] → Additive enum variant with
`#[serde(rename_all = "lowercase")]` tagging already in place; untagged fields default via
`#[serde(default)]` on the containing structs, so old files parse unchanged and simply never
produce the new variant.

[Exclude-set semantics surprise users expecting "exclude" to mean "subtract from current view"
when current view is itself a `Single` or an include set] → Scope to the common case: exclude only
composes against `All` per the spec scenarios; excluding while scoped to `Single` or an include set
falls back to treating it as "exclude from all" (switches to `All` minus the excluded namespace),
documented in the command's tooltip/help text.

[Per-row-only include/exclude limits usefulness for namespaces not currently visible in the table
(e.g. all-namespaces scope already shown, nothing to "add")] → Acceptable for this iteration
(Non-Goals); a namespace picker is the natural follow-up once this lands.
