# Proposal

## Why

Agents currently have no supported route to inspect the clusters connected in Fernrohr, nor to show the user the relevant resource view. An MCP server lets an agent query cluster state and workloads through the app's authenticated sessions, drive the GUI by opening panels and saved layouts, and run a small set of everyday operational actions with the user's approval. It is deliberately not a general-purpose write path: agents cannot create or modify arbitrary resources.

## What Changes

- Add an MCP server hosted by Fernrohr and exposed through a local stdio transport.
- Expose read tools that enumerate connected contexts, discover supported resources, list and get cluster resources, and retrieve pod logs through the selected session.
- Expose tools that open or focus Fernrohr panels for a cluster resource, including the selected namespace and resource identity, and that list and load the user's saved layouts.
- Expose a fixed allowlist of named action tools, each confirmed by the user in the app: set or remove a ConfigMap key, scale a workload, restart a workload's rollout, delete Pods, and trigger a CronJob.
- No tool creates, applies, patches, or deletes arbitrary resources, and none writes Secrets. Return structured, safe errors to MCP clients.

## Capabilities

### New Capabilities

- `agent-mcp`: MCP access to Fernrohr cluster sessions and panel navigation.

### Modified Capabilities

- None.

## Impact

- App process lifecycle and local IPC configuration.
- Cluster session, API discovery, log retrieval, dock panel creation, saved-layout loading, and command routing.
- `resource_actions`: the MCP action tools call the same mutating functions as the in-app actions. Pod delete already exists there; scale and rollout restart are shared with `bulk-select-list-actions`, and CronJob trigger with `resource-specific-actions`. Whichever change lands first adds each function, and the others reuse it.
- A new Rust MCP transport dependency or an in-house protocol adapter, selected during implementation.
