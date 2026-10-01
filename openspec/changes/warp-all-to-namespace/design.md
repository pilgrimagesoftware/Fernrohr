# Design

## Context

Pods already support a focused-panel `WarpNamespace` action. Panel descriptors store namespace scope, while a context owns the open panels used to render a cluster. This change adds shared propagation without changing the existing single-panel behavior.

## Goals / Non-Goals

**Goals:**

- Keep one namespace default per cluster context.
- Update all open namespaced panels in that context in one command.
- Use the same default when opening later namespaced panels.

**Non-Goals:**

- Changing cluster-scoped panels.
- Adding a namespace picker or multi-namespace scope.
- Replacing the existing focused-panel command.

## Decisions

- Store the default on the context/session state, not in each panel, so new panels inherit it and existing panels can be updated together.
- Reuse the existing namespace scope type and command registry. The all-panels command sets `Single(target)` for namespaced panels and leaves cluster-scoped panels unchanged.
- Use the selected Pod's namespace as the target, matching `WarpNamespace`; no new selection UI is needed.

## Risks / Trade-offs

- [A context may contain panels with different namespaces] -> The command intentionally makes them converge to the selected namespace.
- [Persisted state may not contain a default] -> Treat a missing default as unset and retain current defaults.
- [A new panel may open before context state updates] -> Apply the default at panel construction from the same context state used by the command.

## Migration Plan

Add the optional context default field with a serde default. Existing workspace files load with no default; no migration or rollback step is required.
