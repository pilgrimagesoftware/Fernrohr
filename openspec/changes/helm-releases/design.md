# Design

## Context

See proposal.md for why. The specs are `helm-releases`, `helm-release-actions` and the
`resource-browser` delta.

What exists in the App that this builds on:

- **Watches**: `WatchRegistry` refcounts one stream per (context, `WatchKey`), and
  `watch_stream::run_with` takes any `watcher::Config`, including label and field selectors, with
  backoff and 401/403 handling. `WatchKey::Events` with `session/events_watch.rs` is the precedent
  for a typed, non-generic watch.
- **Dedicated panels**: the Events browser (`k8s/resource/events_browser/`) is a non-generic list
  with its own table, columns, filters, sort key, `OpenedPanel::Events`, `NavTarget` routing, a
  restore registration and a `panel_key` arm. Panels persist through their dock `dump`
  (`panel_title::with_saved_scope`) and `register_panel`, not `PanelDescriptor`.
- **Detail building blocks**: `yaml_view` (folding YAML), `SecretValue` (prints only its size),
  `object_detail::redact` (Secret `data`/`stringData` to size placeholders), `resource-links`
  (`ObjectRef`, `viewer_for`, Go to), `confirm_dialog` severities, and the `Refusal` banner.
- **Process spawning**: command tunnels already resolve the login shell's PATH, split arguments
  without a shell, and kill a process group. Tunnels make the connection's effective endpoint differ
  from the kubeconfig's: SSH and forward-mode command tunnels rewrite the server URL to
  `127.0.0.1:<port>` with `tls_server_name` set to the real host, and proxy-mode command tunnels set
  `proxy_url` on the client only.
- **Dependencies**: `base64` and `serde_yaml_ng` are direct dependencies. `flate2` is in the lock
  file transitively only.
- **Tests**: `FakeCluster` serves list, watch, get, merge patch and delete. It ignores label and
  field selectors and has no POST.

## Goals / Non-Goals

**Goals:**

- List releases without downloading every revision's payload.
- Never show, log or keep longer than needed a value or manifest the user hasn't revealed.
- Rollback and Uninstall with exactly Helm's semantics, over any connection Fernrohr can make,
  tunnels included.
- Each tasks section is one reviewable App branch, shipping its own tests and docs.

**Non-Goals:**

- A Rust reimplementation of Helm's release engine.
- Diffing revisions. History shows per-revision values and manifests side by side, not a diff;
  a diff view can follow.
- The `configmap` and `sql` storage drivers, Helm v2, and MCP tools.

## Decisions

### D1. Rows come from a metadata-only watch; payloads are fetched on demand

The list watches Secrets with `labelSelector=owner=helm` and `fieldSelector=type=helm.sh/release.v1`
as **metadata only** (`PartialObjectMeta`), under a new `WatchKey::HelmReleases` per context and
namespace scope, shared by every Helm panel of that context. Helm's labels give each revision's
release `name`, `version` and `status`, so the watch alone yields one row per release (the highest
`version`) and its history list.

Chart, app version, updated time and description need the payload. The table GETs only each
release's **latest** revision Secret, once per new revision (keyed by name and resourceVersion), off
the UI thread. The detail panel GETs other revisions when the History tab first needs them.

*Alternatives:* a full Secret watch decodes everything up front. Each revision is up to 1 MiB
compressed, Helm keeps 10 by default, and every payload would cross a possibly tunneled link on each
relist. The generic Secrets list's `Kind(secrets)` watch can't be reused: it's unfiltered and goes
through the redactor.

### D2. Decoding: Helm's own format, bounded, trimmed

The Secret's `data.release` holds base64 text (Helm's encoding, inside Kubernetes' own base64) of
gzip-compressed JSON. Decode the way Helm does: base64-decode, gunzip if the bytes start with the
gzip magic number, otherwise read the bytes as JSON. Decompression stops at
`HELM_RELEASE_MAX_DECODED_BYTES` (`consts.rs`, 32 MiB), refusing anything larger as unreadable, so a
hostile Secret can't exhaust memory.

Helm 4 writes the same record with extra fields (such as `apply_method`); unknown fields are ignored, so both versions' records decode. Deserialize into a trimmed `HelmRelease`: name, namespace, version, `info` (status, first and last
deployed, description, notes) and chart metadata (name, version, appVersion). `config` (user values)
and `manifest` are kept as `SecretValue`. The chart's templates, files and default values are
skipped, never held. Add `flate2` as a direct dependency, pinned to the version already in the lock
file with a comment, per the project's convention for transitive pins.

### D3. Panels follow the Events browser

A `helm` module under `k8s/resource/helm/`:

- `decode.rs`
- `watch.rs` (the session entry)
- `list/` (panel, table, columns, render, restore, commands)
- `detail/` (panel, tabs, render, restore, commands)
- `manifest.rs` (split, redact, refs)
- `cli/` (find, kubeconfig, run)

`NavTarget::HelmReleases` and `NavTarget::HelmRelease { namespace, name }`, `OpenedPanel::HelmReleases`
and `OpenedPanel::HelmRelease`, `panel_key` arms `"HelmReleases"` and `"HelmRelease"`, and restore
registrations mirror the Events browser. The list reuses the shared column, sort (saved under
`helm-releases`), search, namespace-scope, loading and empty-state machinery. Status tone and icon
use the existing severity palette: `deployed` is good; `failed` is bad; `pending-*` and
`uninstalling` are warning; `superseded` is neutral.

A 403 on the watch shows the access message from the spec instead of an empty table. That message
reuses the generic list's 403 path, worded for Helm.

### D4. Resource panel entry is a non-kind row

Add a `Navigable` row variant for Helm Releases, placed last in Workloads. It's matched by the filter
on "helm", "helm releases" and "releases", opened with Enter or a double-click, and excluded from the
section's kind count. *Alternative rejected:* a synthetic `DiscoveredKind`. It would route through
`Category::for_gvk`, count as a kind, and reach the generic list and detail paths, which would
misrender a release.

### D5. Detail tabs and revision selection

Tabs in this order: Overview, History, Values, Manifest, Notes, Resources. Keys `1`-`6`, the pod
detail panel's tab-key convention, scoped to the panel's key context, each a registered command.

History is a table of revisions, newest first. Its selection is the panel's *selected revision*,
defaulting to the deployed one. Values and Manifest show the selected revision and say which.
History's View Values (`v`) and View Manifest (`m`) commands select a revision and switch tab.

### D6. Hidden by default; redacted even when revealed

Values and Manifest open hidden, with a Reveal control. Its key is `r`, shared by both tabs, which
are never on screen together. Revealed text lives only in the tab's state for the current revision
and is dropped on Hide, on revision change and on close.

Before display, the manifest is split into YAML documents. Every `kind: Secret` document has `data`
and `stringData` rewritten by `object_detail::redact`'s rule. A document that doesn't parse is shown
as "a document that couldn't be read", never raw, so the redaction fails closed. Copy Values and Copy
Manifest are explicit commands, available only while revealed. Nothing from either is logged; the
types make that hard to do by accident, since `SecretValue`'s `Debug` prints only its size.

### D7. Resources tab

The manifest of the *deployed* revision is parsed into `ObjectRef`s: group from `apiVersion`, kind,
name, and namespace defaulting to the release's. The namespace is dropped for kinds discovery reports
as cluster-scoped. Hook resources aren't part of `manifest`, so they aren't listed, which matches
`helm get manifest`. Each row is a `resource-links` reference. Existence is checked by one GET per
object when the tab first opens and on new revisions. A 404 marks the row "missing", and following
it opens the not-found panel, as `resource-links` already specifies. A kind discovery doesn't know
is shown as plain text, per `resource-links`.

### D8. Actions run the user's helm against the connection's effective endpoint (candidate ADR)

Rollback and Uninstall run `helm rollback <name> <revision> -n <ns>` and `helm uninstall <name> -n
<ns>`. Reimplementing them natively would mean reproducing Helm's three-way merge, hook execution
and ordering, wait semantics and release-record bookkeeping. A subtle mismatch there corrupts a
release's history for every other Helm user of that cluster. Linking Helm's Go code would add a Go
toolchain and FFI. Driving the CLI keeps the semantics exact, at the cost of requiring `helm`.

**Finding helm**: a `helm` path in Settings if set, otherwise `helm` on the login shell's PATH (the
command tunnels' resolution). The binary is accepted when `helm version --short` reports `v3.` or `v4.`; both read and write the same `helm.sh/release.v1` storage and take the same rollback and uninstall arguments. The
result is cached per app run and refreshed when Settings change; the actions' availability reads the
cache, never runs a process on a render path.

**Reaching the cluster**: for each run, write a kubeconfig to a fresh file under the cache dir
(created 0600, removed when the run ends, with any leftovers swept at startup). It holds one context
whose cluster is the connection's *effective* endpoint:

- direct: the kubeconfig cluster as is;
- SSH tunnel or forward-mode command tunnel: `server: https://127.0.0.1:<port>` with
  `tls-server-name` set to the real host and the original CA data;
- proxy-mode command tunnel: the real server with `proxy-url` set to the tunnel's local endpoint.

The user stanza is copied from the user's kubeconfig unchanged, including exec plugins, tokens and
client certificates, so `helm` authenticates exactly as the context does. The run gets
`--kubeconfig <file> --kube-context <name>`, the login-shell environment and no shell. The connection
must be Up, or the action is unavailable. The app holds the tunnel for the length of the run, so it
can't be torn down underneath `helm`.

**Running**: off the UI thread, one process group, with a timeout of `HELM_ACTION_TIMEOUT` (10
minutes, `consts.rs`) after which the group is killed and the run reported failed. stdout and stderr
are captured. On failure, the panel's `Refusal` shows stderr's last lines. Success needs no message
of its own; the watch shows the new revision or the release's disappearance. One run per (context,
namespace, release) at a time; a second request while one runs is refused with "already running".

Record this as an App ADR (`docs/adr/`) when section 7 lands.

### D9. Tests without a real cluster or real helm

- **FakeCluster**: honor `labelSelector` (equality terms) and `fieldSelector` (`type=`) on list and
  watch, and answer metadata-only requests (`Accept: ...;as=PartialObjectMetadataList`). These are
  small, general additions other watches can use.
- **Fixtures**: release Secrets generated by a test helper that gzips and encodes a JSON release, plus
  one recorded real-world payload with its credentials removed.
- **helm**: a fake `helm` shell script in a temp dir, set as the Settings path. It records its
  arguments and copies the kubeconfig it was given, so tests assert the effective-endpoint
  kubeconfig for each tunnel kind and the exact arguments. It exits non-zero on demand to test the
  failure path.

## Risks / Trade-offs

- [Requires helm for actions] → Reading never needs it. The actions say why they're disabled and
  where to set the path. The docs page covers installing it.
- [Tunneled endpoint differs per tunnel kind] → One kubeconfig builder with a test per tunnel kind,
  fed from the same resolved `kube::Config` Fernrohr's client uses, not re-derived.
- [Exec-auth plugins need the user's PATH and environment] → Run with the login-shell environment,
  as command tunnels do.
- [Credentials on disk briefly] → 0600 file in the user's own cache dir, removed after the run and
  swept at startup. It holds only what the user's kubeconfig already holds.
- [Large or hostile payloads] → Metadata-only watch; on-demand GETs; decode size cap; trimmed struct.
- [RBAC: listing Secrets is often denied] → An explicit access message per namespace scope. A user
  who may list Secrets in one namespace can scope the panel to it.
- [Revealed values reach the screen] → Per-tab, per-revision reveal, dropped on hide, close and
  revision change. Manifest Secrets are redacted even when revealed.
- [A manifest in an unexpected shape] → Documents that don't parse are withheld, not shown raw; the
  Resources tab lists only what parsed.

## Migration Plan

Additive: new panels, commands, a Resource panel row and a Settings field. No stored state changes.
Workspaces saved before this change restore unchanged. Helm panels saved by a newer build and opened
in an older one fall back to the existing unrestored-panel path.

## Open Questions

- Rollback's and Uninstall's default keys. They're picked during section 8 against the live keymap
  and held by the default-keymap clash test. The specs require a default binding, not a particular
  one.
