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

### Requirement: Allowlisted cluster actions
The MCP server SHALL provide only these state-changing tools: set or remove one key in a ConfigMap's `data`, scale a Deployment, StatefulSet, or ReplicaSet, restart the rollout of a Deployment, StatefulSet, or DaemonSet, delete Pods named explicitly in one namespace, and trigger a Job from a CronJob. Each tool SHALL accept typed fields rather than a resource document, and the server SHALL construct the Kubernetes request so that no field outside the action's scope changes. Before executing an action, Fernrohr SHALL require approval in its user interface for the exact action, context, namespace, kind, every target resource name, and the action's parameters. The tool result SHALL identify whether the action was approved, denied, or failed.

#### Scenario: User approves a scale action
- **WHEN** an agent requests scaling Deployment `api` to 3 replicas and the user approves the displayed action, which shows the current and requested replica counts
- **THEN** Fernrohr sets the replica count through the scale subresource and returns the Kubernetes API result

#### Scenario: User denies a Pod deletion
- **WHEN** an agent requests deletion of named Pods and the user denies the displayed action
- **THEN** Fernrohr does not contact the Kubernetes API and returns a denied result

#### Scenario: Action targets an unsupported kind
- **WHEN** an agent requests a scale action on a DaemonSet, or a ConfigMap value change on a Secret
- **THEN** the MCP response rejects the request without prompting the user or issuing a Kubernetes request

### Requirement: No arbitrary resource writes
The MCP server SHALL NOT provide tools that create, apply, patch, replace, or delete arbitrary resources, edit resource YAML, accept raw Kubernetes API paths, or write Secrets.

#### Scenario: Client lists tools
- **WHEN** an MCP client lists the server's tools
- **THEN** the only state-changing tools listed are the allowlisted cluster actions

### Requirement: Panel navigation tool
The MCP server SHALL provide a tool that opens or focuses a Fernrohr resource panel for a connected context, resource kind, namespace scope, and optional resource name. The tool SHALL return the created or focused panel identifier.

#### Scenario: Open a namespaced resource panel
- **WHEN** an agent requests a Pods panel for context `dev` and namespace `team-a`
- **THEN** Fernrohr opens or focuses a Pods panel scoped to `team-a`

#### Scenario: Open a panel for a resource not already displayed
- **WHEN** an agent requests a panel for a supported resource kind that has no existing panel
- **THEN** Fernrohr creates a panel without changing panels in other windows

### Requirement: Saved layout tools
The MCP server SHALL provide a tool that lists the user's saved layouts by display name, and a tool that loads a named saved layout into the focused window in Add or Replace mode, following the same restore rules as loading it from the app. The MCP server SHALL NOT save, rename, or delete saved layouts.

#### Scenario: Load a saved layout
- **WHEN** an agent requests the saved layout `triage` in Replace mode
- **THEN** Fernrohr replaces the focused window's arrangement with `triage` exactly as the in-app Load Layout command would

#### Scenario: Layout references a context the window does not hold
- **WHEN** a loaded layout contains a panel for a context the window does not currently hold
- **THEN** that panel restores as a placeholder and the MCP response reports it, without connecting the context

### Requirement: Read and navigation tools need no approval
Read, panel navigation, and saved layout tools SHALL execute without a confirmation prompt.

#### Scenario: Agent lists Pods
- **WHEN** an agent lists Pods in a connected context
- **THEN** Fernrohr returns the list without showing a confirmation dialog

### Requirement: Credential and error safety
The MCP server SHALL use only Fernrohr's existing cluster sessions and SHALL never return Kubernetes credentials, kubeconfig contents, authorization headers, tunnel secrets, or internal error details to an MCP client.

#### Scenario: Kubernetes request fails with authentication details
- **WHEN** a Kubernetes API request fails and the upstream error includes authentication material
- **THEN** the MCP response reports a safe failure message without that material
