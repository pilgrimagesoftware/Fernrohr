# Proposal

## Why

Warp to namespace currently changes only the focused panel. During triage, users often need every panel for one cluster context to follow the same namespace, and newly opened panels should keep that choice.

## What Changes

- Add a "Warp all to namespace" command for the selected Pod namespace.
- Apply the target namespace to every open namespaced panel in the active context.
- Store a per-context default namespace used by subsequently opened namespaced panels.
- Preserve the existing single-panel warp command.

## Capabilities

### New Capabilities

- `namespace-warp`: Commands for changing namespace scope across panels and for future panels.

### Modified Capabilities

- `resource-browser`: Namespace selection gains per-context propagation to existing and newly opened panels.

## Impact

- Pod command registration and panel namespace-scope updates in the App submodule.
- Workspace/session state gains a per-context default namespace.
- Existing persisted workspaces remain valid; the default is absent until set.
