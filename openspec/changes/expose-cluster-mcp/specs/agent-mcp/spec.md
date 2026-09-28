# Spec Delta

## Purpose

Allows MCP clients to use Fernrohr's connected cluster sessions and direct the app to show relevant resource panels.

## ADDED Requirements

### Requirement: Local MCP endpoint
Fernrohr SHALL expose an MCP server over a local stdio command that proxies requests to the running Fernrohr application. The proxy SHALL reject requests when no Fernrohr application instance is available and SHALL not expose the service on a network interface.

#### Scenario: Client connects to a running app
- **WHEN** an MCP client launches the Fernrohr MCP command while Fernrohr is running
- **THEN** the client receives the server's tool list and can invoke its tools

#### Scenario: App is unavailable
- **WHEN** an MCP client launches the Fernrohr MCP command while no Fernrohr application instance is running
- **THEN** the command returns a structured unavailable error without starting an application window

### Requirement: Cluster session tools
The MCP server SHALL provide tools that list available contexts, report their connection status, list discovered resource kinds, list resources, retrieve an identified resource, and retrieve pod logs. Each tool SHALL require a context identifier and SHALL return a structured error when the context is unknown, disconnected, or the Kubernetes API rejects the request.

#### Scenario: Agent reads a resource
- **WHEN** an agent requests a Pod by context, namespace, and name from a connected cluster
- **THEN** the MCP response contains that resource's current Kubernetes representation

#### Scenario: Agent requests an unavailable resource kind
- **WHEN** an agent requests a resource kind absent from the selected context's discovery data
- **THEN** the MCP response identifies the unavailable kind and does not issue a Kubernetes request

### Requirement: State-changing cluster operations
The MCP server SHALL provide explicit create, apply, patch, and delete tools for Kubernetes resources. Before executing one of those operations, Fernrohr SHALL require approval in its user interface for the exact operation, context, namespace, kind, and resource name. The tool result SHALL identify whether the operation was approved, denied, or failed.

#### Scenario: User approves an apply operation
- **WHEN** an agent requests an apply operation and the user approves the displayed operation
- **THEN** Fernrohr submits the resource to the selected cluster and returns the Kubernetes API result

#### Scenario: User denies a delete operation
- **WHEN** an agent requests deletion of a resource and the user denies the displayed operation
- **THEN** Fernrohr does not contact the Kubernetes API and returns a denied result

### Requirement: Panel navigation tool
The MCP server SHALL provide a tool that opens or focuses a Fernrohr resource panel for a connected context, resource kind, namespace scope, and optional resource name. The tool SHALL return the created or focused panel identifier.

#### Scenario: Open a namespaced resource panel
- **WHEN** an agent requests a Pods panel for context `dev` and namespace `team-a`
- **THEN** Fernrohr opens or focuses a Pods panel scoped to `team-a`

#### Scenario: Open a panel for a resource not already displayed
- **WHEN** an agent requests a panel for a supported resource kind that has no existing panel
- **THEN** Fernrohr creates a panel without changing panels in other windows

### Requirement: Credential and error safety
The MCP server SHALL use only Fernrohr's existing cluster sessions and SHALL never return Kubernetes credentials, kubeconfig contents, authorization headers, tunnel secrets, or internal error details to an MCP client.

#### Scenario: Kubernetes request fails with authentication details
- **WHEN** a Kubernetes API request fails and the upstream error includes authentication material
- **THEN** the MCP response reports a safe failure message without that material
