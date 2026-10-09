# Design

## Context

Fernrohr owns authenticated `ClusterSession` instances in its primary process, while agent MCP clients commonly start a stdio child command. The server must let that child use the existing sessions and GPUI panel workspace without duplicating credentials or creating a second app process. See [proposal.md](proposal.md) and `agent-mcp` for behavior.

## Goals / Non-Goals

**Goals:**

- Give a local MCP client a stable tool surface for cluster reads, logs, panel and saved-layout navigation, and a small allowlist of approved operational actions.
- Route all Kubernetes work through existing sessions, discovery data, watcher caches, tunnels, and foreground UI dispatch.
- Keep the MCP endpoint local and keep secrets out of tool arguments, responses, and logs.

**Non-Goals:**

- Network-accessible MCP transports, multi-user access, arbitrary shell commands, port-forward control, and streaming watch subscriptions.
- Generic create, apply, patch, or delete of arbitrary resources, YAML editing, and any write to Secrets.
- Saving, renaming, or deleting saved layouts; the agent can only list and load them.
- Bypassing Kubernetes RBAC or user approval for actions.

## Decisions

### Stdio adapter with local app RPC

`fernrohr mcp` will run a stdio MCP adapter. The primary app will create a user-owned Unix-domain socket in its runtime directory and serve a small internal RPC protocol. The adapter forwards validated MCP tool calls to that socket and converts replies to MCP results.

This permits normal MCP client configuration while keeping cluster sessions and UI state in one process. A streamable HTTP endpoint would expose a local network listener and needs authentication and lifecycle handling that this use case does not need. Embedding the stdio server in the GUI process would not work with MCP clients that launch their own command.

### Per-launch local authorization

On startup, the app will generate a random endpoint token, store it with owner-only file permissions beside the socket, and rotate it when the app exits. The adapter reads the token locally and includes it in its internal RPC handshake. The app rejects other users, stale tokens, and malformed requests.

Filesystem placement alone is insufficient on shared machines. Passing the token as a command-line value would expose it to process inspection.

The endpoint is Unix-only: on Windows the app serves nothing and `fernrohr mcp` exits with an explanatory error. The directory is `$XDG_RUNTIME_DIR/fernrohr/mcp` on Linux and the app's cache directory on macOS, which has no per-user runtime directory. The directory is `0700`, and the socket and token are `0600`. The app also checks the connecting peer's uid against its own. A live endpoint is detected by connect-probing the socket rather than with a pidfile; a second app instance keeps no endpoint and logs a warning.

The adapter is built on the official `rmcp` SDK (server and stdio transport only), which owns protocol-version negotiation, JSON-RPC framing, and cancellation. Tools are registered only in the app; the adapter fetches the tool list from the app, so adding a tool never touches the adapter.

### Explicit typed tools

The server will expose a narrow named set of tools: contexts, discovered kinds, list/get resources, pod logs, open panel, list/load saved layouts, and the action tools below. Typed inputs are validated against session discovery before reaching `kube-rs`, and no tool accepts arbitrary API paths or resource documents.

An unrestricted Kubernetes HTTP proxy would be shorter initially but would make authorization, validation, audit logging, and future compatibility harder.

### Allowlisted action tools instead of generic writes

The only state-changing tools are these, each with typed fields rather than a resource document:

| Tool | Targets | Effect |
|---|---|---|
| `set_configmap_value` | ConfigMap | Sets or removes one key in `data` via a merge patch; no other field changes |
| `scale_workload` | Deployment, StatefulSet, ReplicaSet | Sets replicas through the `scale` subresource |
| `restart_workload` | Deployment, StatefulSet, DaemonSet | Rollout restart via the `kubectl.kubernetes.io/restartedAt` pod-template annotation |
| `rollback_workload` | Deployment, StatefulSet, DaemonSet | Rolls back to the previous revision, as `kubectl rollout undo` |
| `set_rollout_paused` | Deployment | Pauses or resumes the rollout by setting `spec.paused` |
| `delete_pods` | Pods, 1 to 10 explicit names in one namespace | Deletes those Pods; owning controllers recreate them |
| `trigger_cronjob` | CronJob | Creates a Job from the CronJob's template, as `kubectl create job --from=cronjob/<name>` |
| `set_cronjob_suspended` | CronJob | Suspends or resumes the schedule by setting `spec.suspend` |

The server builds each request itself from those fields, so an agent cannot change any field outside the action's scope. `delete_pods` takes names rather than a label selector, so the confirmation dialog can list exactly what will be deleted, and it accepts at most 10 names per call. Adding an action later means adding a named tool to this table, not widening an existing one.

Each tool calls the same `resource_actions` function as the matching in-app action (`bulk-select-list-actions` for scale, restart, and rollback; `resource-specific-actions` for CronJob trigger, CronJob suspend/resume, and Deployment pause/resume; the shipped single-row delete for Pods). That way MCP and UI actions behave identically.

A generic patch tool constrained by a field allowlist was considered and rejected: it is harder to show clearly in a confirmation dialog and easier to widen by accident.

### Saved-layout tools

`list_layouts` returns the display names of the user's saved layouts. `load_layout` loads one by name into the focused window in Add or Replace mode, through the same foreground command path as the in-app Load Layout command. Loading follows the `saved-panel-layouts` rules unchanged: a panel whose context the window does not hold restores as a placeholder rather than connecting a context. Saving, renaming, and deleting layouts stay user-only.

### In-app agent setup

A Settings section, Agent access, explains what the MCP server offers, that Fernrohr must be running for it to answer, and that every action asks for approval in the app. It then shows a ready-to-run registration command for each supported harness, each with a copy icon button. A palette command, Copy MCP Setup Command, opens a harness picker and copies the chosen command, so the whole flow is keyboard-reachable.

Each command embeds the absolute, shell-quoted path of the running executable (`std::env::current_exe`, or `$APPIMAGE` when running from an AppImage), so it keeps working regardless of `PATH`. Every command registers at user scope, so the server is available in every project:

| Harness | Command |
|---|---|
| Claude Code | `claude mcp add --scope user fernrohr -- '<exe>' mcp` |
| Codex | `codex mcp add fernrohr -- '<exe>' mcp` |
| Gemini CLI | `gemini mcp add --scope user fernrohr '<exe>' mcp` |
| OpenCode | `opencode mcp add fernrohr --global -- '<exe>' mcp` |

OpenCode's config schema differs between major versions (`mcp.<name>` in v1, `mcp.servers.<name>` in v2), and older releases lack the non-interactive `mcp add` form. Its entry therefore also offers a copyable config snippet, `{"type": "local", "command": ["<exe>", "mcp"]}`, with a note on where it goes. The harness list is one data table, so adding a harness or updating a command syntax touches nothing else.

When the executable path is unstable, the section warns instead of offering a command that will break later. That covers a macOS App Translocation path (the app was opened straight from Downloads or a mounted disk image) and a build-tree path such as `target/debug`. On platforms without the endpoint (Windows), the section says agent access is unavailable and offers no command.

### App-owned confirmation gate

The app routes every action tool through one confirmation request entity. The RPC call waits for an allow or deny response, with a bounded timeout. The dialog names the action, context, namespace, kind, and every target resource name, plus the action's parameters: the ConfigMap key with its old and new values, the old and new replica counts, or the revision a rollback returns to. A denied or timed-out request reaches no Kubernetes API. Read, panel, and layout tools need no confirmation.

MCP client confirmation annotations cannot guarantee an interactive desktop approval and vary by client, so they cannot be the authority for writes.

### UI dispatch boundary

Internal RPC handlers run on the Tokio runtime. Read requests call the selected `ClusterSession`; panel requests cross a bounded channel to the GPUI foreground executor, which opens or focuses a panel through the existing workspace command route. Replies include opaque panel IDs rather than window internals.

Direct access to GPUI state from the RPC task would violate GPUI's thread ownership.

## Risks / Trade-offs

- [MCP adapter starts before the app] -> Return a stable unavailable error and do not launch a GUI.
- [A client disconnects while awaiting confirmation] -> Cancel the pending request and close its dialog when possible; otherwise discard approval and do not execute it.
- [A large resource or log response consumes memory] -> Cap response bytes and return a continuation or truncation indicator.
- [Socket/token files remain after a crash] -> Validate the active app process and token during handshake; remove stale files on next startup.
- [The app is moved or reinstalled to a different path after a harness was registered] -> The harness launches a missing executable and reports it; the Agent access section always shows the current path so the user can re-copy.
- [A panel request targets a closed window] -> Open it in the primary window or return an unavailable UI error when no window exists.

## Migration Plan

1. Add the app-local endpoint disabled only when Fernrohr is not running.
2. Add `fernrohr mcp` and document its MCP client command configuration.
3. Release read tools and panel navigation with fixture-based tests.
4. Release the allowlisted action tools behind the app confirmation gate.
5. Remove endpoint artifacts at app shutdown; retain no migration state.

Rollback removes the CLI MCP command and stops the app endpoint. No cluster resources or persistent format changes are required.
