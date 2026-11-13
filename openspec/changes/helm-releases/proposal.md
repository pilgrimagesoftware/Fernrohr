# Proposal

## Why

Most workloads in the clusters Fernrohr connects to are installed with Helm, but today a release is
invisible: its state lives in `helm.sh/release.v1` Secrets whose payload is gzipped, base64-encoded
JSON, so the generic Secret list shows only opaque names like `sh.helm.release.v1.web.v7`. To answer
"what is deployed, from which chart, with which values, and what changed" the user has to leave
Fernrohr for the `helm` CLI. FreeLens has a Helm view; App#370 asks for one that is first-class and
better: keyboard-first, live, and linked into the rest of the app.

## What Changes

- A **Helm Releases** list panel, opened from the Resource panel and from a "Helm Releases" command:
  one row per release (not per revision) with name, namespace, revision, status, chart, app version
  and last updated, kept live from the cluster, with the standard namespace scope, search and sort.
- A **release detail panel**: an overview (status, chart and app version, revision, first and last
  deployed, description), the **revision history**, the release's **user-supplied values**, the
  rendered **manifest** and the chart's **notes** - each a tab reachable by key - and the
  **resources** the manifest creates, each a link to that object's own panel.
- Values and manifests can hold credentials, so they are **hidden until revealed** the way Secret
  values are, and Secret objects inside a manifest show their keys and sizes, never their data.
- **Rollback** to a chosen earlier revision and **Uninstall**, each behind a confirmation
  (Rollback recoverable, Uninstall irreversible), run by the user's own `helm` CLI against the
  same cluster endpoint Fernrohr's connection uses - tunnel included - so Helm's hooks, three-way
  merge and history bookkeeping behave exactly as they do from a terminal. Without `helm` on the
  PATH the actions are shown disabled, with the reason.
- Every action is a registered command with a key, in the palette and `keymap.toml`.

Out of scope: installing or upgrading charts, chart repositories, the `configmap` and `sql` Helm
storage drivers, Helm v2 (Tiller) releases, and agent (MCP) tools for Helm. Each can follow as its
own change.

## Capabilities

### New Capabilities

- `helm-releases`: discovering a cluster's Helm v3/v4 releases from their storage Secrets, listing them
  live, and showing one release's overview, revision history, values, manifest, notes and created
  resources.
- `helm-release-actions`: rolling a release back to an earlier revision and uninstalling it, through
  the `helm` CLI pointed at the connection's own endpoint, behind confirmations.

### Modified Capabilities

- `resource-browser`: the Resource panel gains a Helm Releases entry that isn't a discovered kind, so
  the requirement that its sections hold exactly the discovered kinds gains that one exception.

## Impact

- App: a `helm` module beside `k8s/resource` (release decoding, the shared release watch, the list and
  detail panels, the CLI runner), a Resource panel entry, new panel descriptors for workspace restore,
  commands and keymap entries. Reuses the shared watch registry, list table machinery, YAML view,
  `SecretValue` hiding, `resource-links`, confirmation dialogs and refusal display.
- Dependencies: a gzip decoder (`flate2`) and a base64 decoder if not already present; no Helm or Go
  library. Rollback and Uninstall need the user's `helm` binary, version 3 or 4.
- Writes a short-lived kubeconfig per CLI run (permissions 0600, deleted after) describing the
  connection's effective endpoint; credentials are never logged. A candidate ADR: driving `helm`
  rather than reimplementing its rollback and uninstall.
- Docs: a new `docs/site/latest/helm.md` page (with its `_data/versions.yml` entry and sidebar nav),
  the Resource panel section of `resources.md`, and `reference/keybindings.md` for the new commands.
