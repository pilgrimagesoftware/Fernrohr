# Design

## Context

Resource-level commands today (`pods.describe`, `pods.logs`, `pods.yaml`) are each a small
`Command` registered per resource-kind module (`k8s/resource/pods/commands.rs`,
`pod_detail/commands.rs`, `object_detail/commands.rs`) that calls into that kind's existing panel
state. This change adds destructive and interactive commands (delete, kill, edit, exec,
port-forward) in the same style, plus two cross-cutting commands (help overlay, namespace
quick-jump) that read from the command registry and the panel's existing namespace list rather than
a resource kind's state.

Exec and port-forward both need a long-lived, bidirectional connection from a background tokio task
into a GPUI view - the same shape `managed-forward`'s Kubernetes implementation and `pod-logs`'
streaming already use (kube-rs async stream → bounded channel → GPUI Entity).

## Goals / Non-Goals

**Goals:**
- Reuse the existing command registry, confirmation-dialog, and resource-row-action wiring rather
  than introducing a second mechanism for "commands that act on a resource."
- Reuse `managed-forward` for port-forward instead of a separate port-forward implementation.
- Keep destructive actions (delete, kill) requiring an explicit, resource-kind-agnostic
  confirmation path so new resource kinds don't each reinvent one.

**Non-Goals:**
- A full terminal multiplexer or multi-pane shell experience - shell-exec is one panel, one session,
  no split panes or tabs within it.
- Editing resources other than via raw YAML (no structured/form-based editor).
- k9s features not in this proposal's scope (resource-alias toggle, wide/decor view, `:` command
  mode) - these are either low value for a GUI app or better scoped separately.

## Decisions

**Delete/kill as one command pair sharing a backend call, not two unrelated features.** Both send a
`DeleteParams` to the same typed delete API; kill just sets `grace_period_seconds(0)` and skips the
confirmation dialog. One small `resource_actions::delete(client, kind, name, ns, force: bool)`
helper, reused by both commands and by any resource kind that registers them. Alternative
considered: separate kill-specific API call - rejected, Kubernetes's delete API already supports
zero grace period.

**Exec reuses the log panel's streaming shape, not a new transport.** `pod-logs` already proves
kube-rs async stream → bounded channel → GPUI Entity for one-directional data. Exec needs the same
pipe plus a reverse channel for keystrokes into `AttachParams`' stdin. Alternative considered: a
dedicated PTY/terminal crate - rejected for this change; the exec session is a scrollback + input
line, not a full terminal emulator, so no PTY allocation is needed on the Fernrohr side (the
container's own shell handles line discipline).

**Edit-YAML reuses `object_detail`'s existing YAML rendering, made editable.** `object_detail`
already renders a resource as YAML (`object_detail.toggle_view`). Edit mode is that same rendering
made writable, with save doing a `Patch::Apply` (server-side apply) so a resourceVersion conflict
surfaces as a normal apply error rather than requiring manual conflict resolution. Alternative
considered: a diff-based three-way merge UI - rejected as disproportionate for a first version;
server-side apply already gives a clear conflict signal to extend later if needed.

**Port-forward-from-a-row acquires a `managed-forward` directly, bypassing pre-configured tunnel
entries.** The row command builds a Kubernetes-target `ManagedForward` acquire request the same way
the tunnel-management UI does when a user adds one by hand, so it appears in that same list and
follows the same reference-counted lifecycle. Alternative considered: a separate "ad hoc forward"
list scoped to the panel - rejected, it would duplicate the lifecycle and state `managed-forward`
already owns.

**Help overlay is read-only and derived, not a second source of command metadata.** It walks the
same `CommandRegistry` the palette and keymap already read, filtered by the currently focused
`KeyContext`, so a new command is automatically listed without a second registration.

**Namespace quick-jump binds to existing numeric key slots already in use elsewhere
(`tab.select_1`-`9` use `ctrl-1`-`9`).** Quick-jump commands use a different modifier combination
scoped to the resource panel's `KeyContext` (not global), so they don't collide with tab selection;
exact keys are a keymap default to pick during implementation, validated by the existing keymap
conflict checker (`keymap::conflicts`).

## Risks / Trade-offs

- [Exec sessions holding an open connection per Pod] → `managed-forward`'s reference-counted
  shutdown pattern already handles "last holder closes, transport tears down"; exec sessions use the
  same close-on-panel-close path, so no separate cleanup mechanism is needed.
- [Delete/kill is destructive and irreversible] → delete requires explicit confirmation by default;
  kill is scoped to Pods (not arbitrary resource kinds) in the initial implementation, since
  "force-kill" is primarily a Pod-restart workflow in k9s.
- [Edit-YAML's server-side apply can silently take ownership of fields another controller manages] →
  surfacing the apply result's `managedFields` conflicts as the failure case (not a silent merge) is
  required by `resource-actions`' "Server rejects the update" scenario; this is a known
  server-side-apply trade-off, not unique to Fernrohr.
- [Numeric quick-jump binding choice might surprise k9s users expecting bare `0`-`9`] → bare digits
  are unavailable globally (conflict with text input in filters and edit views), so Fernrohr's
  binding will necessarily differ from k9s's; documented via the help overlay rather than matched
  exactly.

## Open Questions

- Exact default key bindings for the new commands (delete, kill, edit, shell, port-forward,
  quick-jump, help overlay) - left to `tasks.md` to assign, checked against `keymap::conflicts`
  during implementation rather than decided here.
