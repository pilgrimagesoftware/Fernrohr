# agent-mcp Specification

## Purpose
Allows MCP clients to use Fernrohr's connected cluster sessions and direct the app to show relevant resource panels.

## Requirements

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
The MCP server SHALL provide only these state-changing tools: set or remove one key in a ConfigMap's `data`, scale a Deployment, StatefulSet, or ReplicaSet, restart or roll back to the previous revision the rollout of a Deployment, StatefulSet, or DaemonSet, pause or resume a Deployment's rollout, delete between 1 and 10 Pods named explicitly in one namespace, trigger a Job from a CronJob, and suspend or resume a CronJob.

#### Scenario: Action targets an unsupported kind
- **WHEN** an agent requests a scale action on a DaemonSet, a rollout pause on a StatefulSet, or a ConfigMap value change on a Secret
- **THEN** the MCP response rejects the request without prompting the user or issuing a Kubernetes request

#### Scenario: Pod deletion exceeds the limit
- **WHEN** an agent requests deletion of 11 or more Pods in one call
- **THEN** the MCP response rejects the request without prompting the user or issuing a Kubernetes request

### Requirement: Typed action requests
Each action tool SHALL accept typed fields rather than a resource document, and the server SHALL construct the Kubernetes request so that no field outside the action's scope changes.

#### Scenario: ConfigMap value change touches one key
- **WHEN** an agent sets key `LOG_LEVEL` in a ConfigMap
- **THEN** the request Fernrohr sends changes only that key in `data`

### Requirement: Action approval
Before executing an action, Fernrohr SHALL require approval in its user interface for the exact action, context, namespace, kind, every target resource name, and the action's parameters. The tool result SHALL identify whether the action was approved, denied, or failed.

#### Scenario: User approves a scale action
- **WHEN** an agent requests scaling Deployment `api` to 3 replicas and the user approves the displayed action, which shows the current and requested replica counts
- **THEN** Fernrohr sets the replica count through the scale subresource and returns the Kubernetes API result

#### Scenario: User denies a Pod deletion
- **WHEN** an agent requests deletion of named Pods and the user denies the displayed action
- **THEN** Fernrohr does not contact the Kubernetes API and returns a denied result

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

### Requirement: In-app agent setup instructions
Fernrohr SHALL provide an Agent access section in Settings that describes the MCP server, states that Fernrohr must be running for it to respond and that actions require in-app approval, and shows a registration command for at least Claude Code, Codex, Gemini CLI, and OpenCode. Each command SHALL invoke the running installation's absolute executable path with the `mcp` argument and register the server at user scope.

#### Scenario: Commands use the installed executable
- **WHEN** the user opens the Agent access section
- **THEN** each harness's command runs this installation's absolute executable path with the `mcp` argument

### Requirement: Copying agent setup commands
Each harness's registration command SHALL be copyable to the clipboard with an icon button and through a command-palette command. For OpenCode, the section SHALL also offer a copyable configuration snippet.

#### Scenario: Copy a harness command
- **WHEN** the user activates the copy button for Claude Code in the Agent access section
- **THEN** the clipboard holds a `claude mcp add` command that runs this installation's executable with the `mcp` argument at user scope

#### Scenario: Copy from the keyboard
- **WHEN** the user runs Copy MCP Setup Command from the command palette and picks Codex
- **THEN** the clipboard holds the Codex registration command without the user touching the mouse

#### Scenario: Unstable executable path
- **WHEN** Fernrohr is running from a macOS translocated path or a build directory
- **THEN** the Agent access section warns that the path will not persist and explains how to install the app before copying

#### Scenario: Platform without the endpoint
- **WHEN** Fernrohr runs on a platform where the MCP endpoint is unavailable
- **THEN** the Agent access section states that agent access is unavailable and offers no command

### Requirement: Credential and error safety
The MCP server SHALL use only Fernrohr's existing cluster sessions and SHALL never return Kubernetes credentials, kubeconfig contents, authorization headers, tunnel secrets, or internal error details to an MCP client.

#### Scenario: Kubernetes request fails with authentication details
- **WHEN** a Kubernetes API request fails and the upstream error includes authentication material
- **THEN** the MCP response reports a safe failure message without that material

### Requirement: Connect context tool
The MCP server SHALL provide a tool that connects a kubeconfig context in the frontmost Fernrohr window, adding it to that window as the status bar's add-context control does, or opening a window for it when none is open. A context the window already holds SHALL be focused without connecting again.

#### Scenario: Connect a context that is not open
- **WHEN** an agent asks to connect `staging` and the user approves
- **THEN** the frontmost window gains `staging`, opens its Pods panel, and makes it the selected cluster

#### Scenario: Context already in the window
- **WHEN** an agent asks to connect a context the frontmost window already holds
- **THEN** no second connection is made and no approval is asked, and the result reports its current state

#### Scenario: Unknown context
- **WHEN** an agent asks to connect a name that is not in the kubeconfig
- **THEN** the MCP response is an unknown-context error and nothing is asked or connected

### Requirement: Connecting requires approval
Before connecting a context the window does not already hold, Fernrohr SHALL ask the user to approve, naming the context and its bound tunnel if any. A denied, timed-out, or abandoned request SHALL connect nothing.

#### Scenario: User denies a connection
- **WHEN** an agent asks to connect `prod` and the user denies it
- **THEN** no connection, tunnel, or credential flow starts, and the result is denied

### Requirement: Connect reports the settled state
After an approved connection starts, the tool SHALL wait up to a bounded time for it to settle and SHALL report connected, waiting for its tunnel, awaiting a manual tunnel's confirmation, failed with a safe reason, or still connecting when the wait ends.

#### Scenario: Manual tunnel needs the user
- **WHEN** an agent connects a context bound to a manual tunnel that is not yet confirmed
- **THEN** the result reports that it is awaiting confirmation, and the user's prompt is shown as usual

#### Scenario: Slow connection
- **WHEN** a connection has not settled when the wait ends
- **THEN** the result reports it as still connecting, and the agent can check it later with the context status tool

### Requirement: UI tools bring the app to the front
Every tool that changes what Fernrohr shows SHALL bring the application to the front and focus the window it acted on. Tools that only read SHALL NOT change focus.

#### Scenario: Open a panel from the terminal
- **WHEN** an agent opens a Pods panel while the user's terminal is the active application
- **THEN** Fernrohr becomes the active application with the window holding that panel focused

#### Scenario: Reads stay in the background
- **WHEN** an agent lists resources or saved layouts while Fernrohr is in the background
- **THEN** Fernrohr stays in the background
