# Proposal

## Why

Agents currently have no supported route to inspect or operate the clusters connected in Fernrohr, nor to show the user the relevant resource view. An MCP server lets an agent use the app's authenticated cluster sessions and open focused panels for review.

## What Changes

- Add an MCP server hosted by Fernrohr and exposed through a local stdio transport.
- Expose tools that enumerate connected contexts, discover supported resources, read cluster resources and logs, and perform permitted resource operations through the selected session.
- Expose tools that open or focus Fernrohr panels for a cluster resource, including the selected namespace and resource identity.
- Require an explicit operation model for state-changing cluster requests and return structured, safe errors to MCP clients.

## Capabilities

### New Capabilities

- `agent-mcp`: MCP access to Fernrohr cluster sessions and panel navigation.

### Modified Capabilities

- None.

## Impact

- App process lifecycle and local IPC configuration.
- Cluster session, API discovery, resource operations, log retrieval, dock panel creation, and command routing.
- A new Rust MCP transport dependency or an in-house protocol adapter, selected during implementation.
