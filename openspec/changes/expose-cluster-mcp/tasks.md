# Tasks

## 1. Local endpoint and protocol

- [ ] 1.1 Add app lifecycle support for a user-owned Unix socket, rotating token file, stale-file cleanup, and authenticated internal RPC handshake; verify socket integration tests reject missing and stale tokens.
- [ ] 1.2 Add the `fernrohr mcp` stdio command and MCP protocol adapter; verify an MCP client fixture lists tools through a running app and receives an unavailable error without one.
- [ ] 1.3 Define typed RPC and MCP request and response models, size limits, safe error mapping, and structured logging that excludes secrets; verify serialization and error-redaction unit tests pass.

## 2. Cluster MCP tools

- [ ] 2.1 Implement context status and discovered-resource-kind tools backed by `ClusterRegistry` and `ClusterSession`; verify fixture sessions return only their own discovery data.
- [ ] 2.2 Implement list and get resource tools with context, namespace, and discovery validation; verify unknown contexts and unsupported kinds do not reach the Kubernetes client.
- [ ] 2.3 Implement bounded pod-log retrieval and resource response truncation behavior; verify an oversized fixture response reports truncation without exceeding the configured limit.

## 3. Approved cluster writes

- [ ] 3.1 Implement typed create, apply, patch, and delete tools through the selected `ClusterSession`; verify request validation rejects malformed target and resource documents.
- [ ] 3.2 Add the foreground confirmation gate with operation details, cancel handling, and timeout; verify denial, timeout, and client-disconnect tests make no Kubernetes write call.
- [ ] 3.3 Connect approved write operations to `kube-rs` and safe MCP results; verify approved fixture operations return the API result and redact upstream authentication data.

## 4. Panel navigation

- [ ] 4.1 Add the open-or-focus panel RPC route through the existing command and workspace path; verify requests cross from the Tokio handler to the GPUI foreground executor.
- [ ] 4.2 Support context, kind, namespace scope, and optional resource selection in panel requests; verify panel integration tests preserve panels in other windows and return an opaque panel ID.

## 5. Integration checks

- [ ] 5.1 Add MCP client integration coverage for reads, writes, denied writes, panel navigation, and unavailable app behavior using recorded Kubernetes fixtures; verify `cargo test` passes without a live cluster.
- [ ] 5.2 Document MCP client command configuration and user approval behavior; verify `cargo fmt -- --check`, `cargo clippy -- -D warnings`, and `cargo test` pass.
