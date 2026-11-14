# Tasks

Each numbered section is one App branch and PR (`370-helm-<section>`), landing its own tests and
docs. Every PR passes `cargo fmt --check`, `cargo clippy --all-targets -- -D warnings` and
`cargo test`.

## 1. Release decoding

- [x] 1.1 Add `flate2` as a direct dependency, pinned to the lock file's transitive version with a comment, and `HELM_RELEASE_MAX_DECODED_BYTES` to `consts.rs`. Verify with `cargo tree -i flate2` showing one version.
- [x] 1.2 Add `k8s/resource/helm/decode.rs`: Helm's payload format (base64, then gzip when it starts with the magic number, else raw JSON) into the trimmed `HelmRelease` of design D2, with values and manifest held as `SecretValue`. Verify with unit tests: a gzipped fixture, an uncompressed one, a payload over the cap refused, malformed base64/gzip/JSON each an error, and the chart's templates absent from the decoded struct. Include one recorded real payload with its credentials removed.

## 2. Release watch

- [x] 2.1 Extend `FakeCluster` to honor equality `labelSelector` terms and `fieldSelector=type=` on list and watch, and to answer metadata-only list and watch requests. Verify with FakeCluster tests for each, and that existing tests still pass.
- [x] 2.2 Add `WatchKey::HelmReleases` and its session entry: a metadata-only, selector-filtered Secret watch per context and namespace scope, grouping revisions by (namespace, release name) with the latest by `version`, and fetching and decoding each release's latest revision once per new revision, off the UI thread (D1). Verify with tests over FakeCluster: install (v1), upgrade (v2 replaces the row's revision), uninstall (row gone), two subscribers sharing one stream, a corrupt latest revision marked unreadable while others list, and a 403 surfaced as an access refusal.

## 3. Helm Releases list

- [x] 3.1 Add the Helm Releases list panel per the `helm-releases` list requirement: columns, status tone and icon (D3), namespace scope, search, sort saved under `helm-releases`, loading, empty and access-refused states, `NavTarget::HelmReleases`, `OpenedPanel::HelmReleases`, workspace restore, and a "Helm Releases" command. Verify with window tests for live upgrade and uninstall, the access message under all namespaces, a restore round trip, and a keystroke-only open from the palette.
- [x] 3.2 Add the Resource panel's Helm Releases row last in Workloads (D4), excluded from the kind count, filterable and openable by Enter or double-click. Verify with the `resource-browser` delta's scenarios as tests, including that the kind count still equals the discovered kinds.
- [x] 3.3 Docs: add `docs/site/latest/helm.md` (the list, its columns and statuses, the access message), with its `_data/versions.yml` pages and nav entry. Mention Helm Releases in `resources.md`'s Resource panel section, and add the new commands to `reference/keybindings.md`. Placeholders only. Verify with the docs site checks in `cargo test`.

## 4. Release detail: Overview, History, Notes

- [x] 4.1 Add the release detail panel: `NavTarget::HelmRelease`, opened by Enter, double-click and `d` from the list, Overview, History (revisions newest first, deployed marked, revision headers fetched on first need) and Notes tabs, tab keys `1`-`6` as registered commands (D5), a live update on new revisions, a "release no longer exists" state, and workspace restore. Verify with window tests: open by keys, History after an upgrade, the gone state after uninstall, and a restore round trip.
- [x] 4.2 Docs: the detail panel's Overview, History and Notes in `helm.md`, and the tab keys in `reference/keybindings.md`. Verify with the docs site checks.

## 5. Values and Manifest

- [x] 5.1 Add the Values and Manifest tabs: hidden until revealed (`r`), per tab and per revision, dropped on hide, revision change and close; History's View Values (`v`) and View Manifest (`m`) select a revision (D5, D6); Copy Values and Copy Manifest only while revealed. Verify with tests: tabs open hidden, reveal and hide by key, the reveal dropped on revision change, a revision-3 view stating its revision, and Copy unavailable while hidden.
- [x] 5.2 Manifest redaction (D6): split documents, redact every Secret's `data` and `stringData` to key and size, and withhold documents that don't parse. Verify with unit tests for a Secret with both fields, a multi-document manifest, and an unparseable document never shown raw.
- [x] 5.3 Docs: Values and Manifest in `helm.md`, including that Secrets in a manifest show only sizes, and their keys in `reference/keybindings.md`. Verify with the docs site checks.

## 6. Resources

- [x] 6.1 Add the Resources tab (D7): the deployed revision's manifest as `ObjectRef` rows with namespace defaulting and cluster-scoped kinds unnamespaced, each a `resource-links` reference, an existence check marking missing objects, and plain text for kinds discovery doesn't know. Verify with tests: following a Deployment opens its panel, a missing ConfigMap is marked and opens the not-found panel, and a ClusterRole is shown without a namespace.
- [x] 6.2 Docs: the Resources tab in `helm.md`. Verify with the docs site checks.

## 7. helm CLI runner

- [x] 7.1 Find `helm` (D8): a new Settings field for its path, else the login shell's PATH, accepted when `helm version --short` reports v3, cached and refreshed on a Settings change. Verify with tests over a fake `helm` script: found on PATH, found at the Settings path, v2 or missing rejected with the reason.
- [x] 7.2 Build the per-run kubeconfig from the connection's resolved endpoint (D8): direct, SSH tunnel, forward-mode and proxy-mode command tunnels, with the user stanza copied unchanged, written 0600, removed after the run, and swept at startup. Verify with a unit test per connection kind asserting the server, `tls-server-name` and `proxy-url`, a test that the file's mode is 0600, and one that it is gone after the run.
- [x] 7.3 Run `helm` off the UI thread with no shell, in its own process group, with the login-shell environment, `HELM_ACTION_TIMEOUT`, captured output, one run per release, and the tunnel held for the run's length. Verify with fake-`helm` tests: arguments as expected, a non-zero exit reported with stderr, a timeout killing the group, a second run on the same release refused.
- [x] 7.4 Write the App ADR for driving `helm` instead of reimplementing its release engine (D8), in `docs/adr/`. Verify it is linked from the ADR index.
- [x] 7.5 Docs: in `helm.md`, a "Requirements" note saying the actions need Helm 3 and how to set its path, and the Settings page for the new field. Verify with the docs site checks.

## 8. Rollback and Uninstall

- [x] 8.1 Add Rollback per `helm-release-actions`: from History's selected revision (default: the one before current) and as a command, behind a recoverable confirmation naming release, namespace, current and target revisions with chart and app version, running `helm rollback`. Verify with fake-`helm` window tests: a keystroke-only rollback to revision 3 with the expected arguments, Escape running nothing, and a failure shown with `helm`'s message.
- [x] 8.2 Add Uninstall from the list's row (key and context menu) and the detail panel, behind an irreversible confirmation naming the release and namespace and counting the manifest's objects, focus on Cancel, Enter cancelling, the deliberate shortcut confirming, running `helm uninstall`. Verify with fake-`helm` window tests for Enter cancelling and the shortcut confirming with the expected arguments.
- [x] 8.3 Disabled states: with `helm` missing, or the connection not Up, both actions show as disabled in menus, buttons and the palette, with the reason. A running action shows progress and refuses a second. Verify with tests for each state, and pick default keys held by `no_default_binding_clashes`.
- [x] 8.4 Docs: Rollback and Uninstall in `helm.md` (what each runs, the confirmations, tunneled contexts), and their keys in `reference/keybindings.md`. Verify with the docs site checks.

## 9. Integration

- [x] 9.1 Manual check on a real cluster through a tunneled context: list releases, open one, read History, reveal Values, follow a resource, roll back to an earlier revision, and uninstall a throwaway release. Record the date and result here.
  Waived by the maintainer (2026-11-13): covered by the local minikube integration run (2026-11-13 18:40 UTC, App#409: list, history, values reveal, resources, rollback and uninstall through the real helm runner) plus the per-tunnel-kind kubeconfig unit tests (App#401).

## Workflow follow-up

- Archive the change (`openspec archive helm-releases`) after section 9 passes, syncing the
  `helm-releases`, `helm-release-actions` and `resource-browser` specs.
