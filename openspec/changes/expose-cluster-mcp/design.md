# Design

## Context

Fernrohr owns authenticated `ClusterSession` instances in its primary process, while agent MCP clients commonly start a stdio child command. The server must let that child use the existing sessions and GPUI panel workspace without duplicating credentials or creating a second app process. See [proposal.md](proposal.md) and `agent-mcp` for behavior.

## Goals / Non-Goals

**Goals:**

- Give a local MCP client a stable tool surface for cluster reads, approved writes, logs, and panel navigation.
- Route all Kubernetes work through existing sessions, discovery data, watcher caches, tunnels, and foreground UI dispatch.
- Keep the MCP endpoint local and keep secrets out of tool arguments, responses, and logs.

**Non-Goals:**

- Network-accessible MCP transports, multi-user access, arbitrary shell commands, port-forward control, and streaming watch subscriptions.
- Bypassing Kubernetes RBAC or user approval for writes.

## Decisions

### Stdio adapter with local app RPC

`fernrohr mcp` will run a stdio MCP adapter. The primary app will create a user-owned Unix-domain socket in its runtime directory and serve a small internal RPC protocol. The adapter forwards validated MCP tool calls to that socket and converts replies to MCP results.

This permits normal MCP client configuration while keeping cluster sessions and UI state in one process. A streamable HTTP endpoint would expose a local network listener and needs authentication and lifecycle handling that this use case does not need. Embedding the stdio server in the GUI process would not work with MCP clients that launch their own command.

### Per-launch local authorization

On startup, the app will generate a random endpoint token, store it with owner-only file permissions beside the socket, and rotate it when the app exits. The adapter reads the token locally and includes it in its internal RPC handshake. The app rejects other users, stale tokens, and malformed requests.

Filesystem placement alone is insufficient on shared machines. Passing the token as a command-line value would expose it to process inspection.

### Explicit typed tools

The server will expose a narrow named set of tools: contexts, discovered kinds, list/get resources, pod logs, create/apply/patch/delete resources, and open panel. Typed inputs are validated against session discovery before reaching `kube-rs`; resource writes accept a resource document and target context rather than arbitrary API paths.

An unrestricted Kubernetes HTTP proxy would be shorter initially but would make authorization, validation, audit logging, and future compatibility harder.

### App-owned confirmation gate

The app routes all writes through one confirmation request entity. The RPC call waits for an allow or deny response, with a bounded timeout. The dialog names the action, context, namespace, kind, and resource name; it never renders secret content. A denied or timed-out request reaches no Kubernetes API.

MCP client confirmation annotations cannot guarantee an interactive desktop approval and vary by client, so they cannot be the authority for writes.

### UI dispatch boundary

Internal RPC handlers run on the Tokio runtime. Read requests call the selected `ClusterSession`; panel requests cross a bounded channel to the GPUI foreground executor, which opens or focuses a panel through the existing workspace command route. Replies include opaque panel IDs rather than window internals.

Direct access to GPUI state from the RPC task would violate GPUI's thread ownership.

## Risks / Trade-offs

- [MCP adapter starts before the app] -> Return a stable unavailable error and do not launch a GUI.
- [A client disconnects while awaiting confirmation] -> Cancel the pending request and close its dialog when possible; otherwise discard approval and do not execute it.
- [A large resource or log response consumes memory] -> Cap response bytes and return a continuation or truncation indicator.
- [Socket/token files remain after a crash] -> Validate the active app process and token during handshake; remove stale files on next startup.
- [A panel request targets a closed window] -> Open it in the primary window or return an unavailable UI error when no window exists.

## Migration Plan

1. Add the app-local endpoint disabled only when Fernrohr is not running.
2. Add `fernrohr mcp` and document its MCP client command configuration.
3. Release read tools and panel navigation with fixture-based tests.
4. Release write tools behind the app confirmation gate.
5. Remove endpoint artifacts at app shutdown; retain no migration state.

Rollback removes the CLI MCP command and stops the app endpoint. No cluster resources or persistent format changes are required.
