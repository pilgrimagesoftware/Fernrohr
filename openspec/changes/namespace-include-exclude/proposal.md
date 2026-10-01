# Proposal

## Why

Namespace scoping today is binary per panel: a single namespace or all namespaces
(`resource-browser`'s "Namespace scoping" requirement, backed by `NamespaceScope::{All, Single}`).
Pods also has a one-off "warp to namespace" command that re-scopes the panel to the selected pod's
namespace. Neither covers the common triage case: watching a handful of namespaces together (e.g.
every namespace a team owns) or excluding noisy system namespaces (`kube-system`, `kube-public`)
while otherwise watching everything. Users currently have to flip one panel at a time between
"All" and a single namespace, losing the rest of the set each time.

## What Changes

- Add an include/exclude namespace set alongside the existing `All` / `Single` scope, so a panel
  can watch "these namespaces" or "all except these namespaces."
- Add "Include Namespace" / "Exclude Namespace" commands that add or remove the selected row's (or
  a chosen) namespace from the current panel's set, without discarding the rest of the set the way
  `NamespaceScope::Single` does.
- Add "Include Namespace for All" / "Exclude Namespace for All" commands that apply the same
  add/remove to every open panel in the window's active context(s), and set that context's default
  namespace scope so panels opened afterward in that context start with the updated set - mirroring
  the propagation model introduced by `warp-all-to-namespace`.
- **BREAKING**: `NamespaceScope`'s on-disk shape gains a variant (`IncludeExclude`); older
  `workspace.toml` files with only `All`/`Single` still parse unchanged (additive enum).

## Capabilities

### Modified Capabilities
- `resource-browser`: "Namespace scoping" requirement extends from a single-namespace-or-all
  choice to an include/exclude namespace set, plus per-context default propagation to new and
  existing panels.

## Impact

- `App/app/src/config/workspace.rs`: `NamespaceScope` gains an `IncludeExclude { mode, namespaces }`
  variant (`mode` = Include or Exclude).
- `App/app/src/k8s/resource/pods/commands.rs` and `pods.rs`: new commands, replacing or
  complementing the existing single-shot `WarpNamespace`.
- Per-context default namespace scope storage (introduced by `warp-all-to-namespace`): this change
  reuses that mechanism rather than inventing a second one.
- Any resource-kind panel beyond Pods that adopts namespace scoping inherits the same set semantics
  for free, since it lives in the shared `NamespaceScope` type.
