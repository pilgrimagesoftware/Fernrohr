# helm-releases Specification

## Purpose
Show a cluster's Helm v3/v4 releases, read from Helm's own storage in the cluster, as a live list and a
per-release detail view of its status, history, values, manifest, notes and created resources.

## Requirements

### Requirement: Releases are read from Helm's Secret storage

The application SHALL discover a context's Helm releases from the Secrets Helm v3 and v4 store them in by
default: type `helm.sh/release.v1`, labelled `owner=helm`, one Secret per revision. It SHALL decode a
revision's payload (gzip-compressed JSON, base64-encoded inside the Secret's data) without running
`helm`. A Secret that claims to be a release but can't be decoded SHALL NOT stop other releases from
showing; its release SHALL show that its revision couldn't be read.

#### Scenario: A release with several revisions

- **WHEN** a namespace holds `sh.helm.release.v1.web.v1` through `.v3` for release `web`
- **THEN** the application knows release `web` in that namespace at revision 3, with revisions 1-3 in
  its history

#### Scenario: A corrupt revision

- **WHEN** one release's latest revision Secret holds data that doesn't decode
- **THEN** every other release still lists, and that release's row says its latest revision couldn't
  be read

### Requirement: Helm Releases list

The application SHALL offer a Helm Releases list panel for a cluster context, opened from a Helm
Releases entry in the Resource panel and from a registered "Helm Releases" command. It SHALL show
one row per release, not per revision, with Name, Namespace, Revision, Status, Chart (name and
version), App Version and Updated.

#### Scenario: Opened from the palette

- **WHEN** the user runs "Helm Releases" from the command palette in a connected window
- **THEN** the Helm Releases list opens for the window's active context

### Requirement: The release list stays live

The Helm Releases list SHALL reflect installs, upgrades, rollbacks and uninstalls without the user
refreshing.

#### Scenario: An upgrade appears without refreshing

- **WHEN** a release is upgraded from revision 4 to 5 while its list is open
- **THEN** its row shows revision 5, its new status and its new chart version, with no other row
  added

#### Scenario: Uninstalled without history

- **WHEN** a release is uninstalled and its revision Secrets are deleted
- **THEN** its row disappears

### Requirement: The release list behaves like every list panel

The Helm Releases list SHALL offer the namespace scope, search, sort, and loading and empty states
every list panel offers, and SHALL tell release statuses apart without relying on color.

#### Scenario: Statuses are told apart without color

- **WHEN** releases are `deployed`, `failed`, `pending-upgrade` and `uninstalling`
- **THEN** each row's status reads as that word, and failed and pending ones are also marked by tone
  and icon

### Requirement: Access to release storage

When the user may not list Secrets in the panel's namespace scope, the Helm Releases list SHALL say
so, naming the namespace scope, instead of showing an empty list.

#### Scenario: Secrets are forbidden cluster-wide

- **WHEN** the list is scoped to all namespaces and listing Secrets cluster-wide is forbidden
- **THEN** the list says it may not read Helm's storage across all namespaces and suggests choosing
  a namespace

### Requirement: Release detail panel

Opening a release from its row (Enter, double-click, or describe `d`) SHALL open a release detail
panel for that release in the same context, showing an Overview, History, Values, Manifest, Notes
and Resources tab. Each tab SHALL be reachable by its own key and by click, and the panel SHALL
follow the release live: a new revision updates every tab. A release that no longer exists SHALL be
shown as gone, not as an error.

#### Scenario: Overview of a deployed release

- **WHEN** the detail panel opens for a deployed release
- **THEN** its Overview shows name, namespace, status, revision, chart name and version, app
  version, first deployed, last deployed and the last operation's description

#### Scenario: History

- **WHEN** the History tab is shown for a release at revision 5
- **THEN** it lists revisions 5 to 1, newest first, each with its updated time, status, chart, app
  version and description, and the deployed one marked

#### Scenario: The release is uninstalled while open

- **WHEN** a release's last revision Secret is deleted while its detail panel is open
- **THEN** the panel says the release no longer exists

### Requirement: Values and manifest stay hidden until revealed

A revision's user-supplied values and its rendered manifest SHALL NOT be shown until the user reveals
them, per tab and per revision, and SHALL be hidden again when the user hides them or the panel
closes. Even when revealed, a Secret object inside the manifest SHALL show its keys and each value's
size, never its data. Neither SHALL be written to logs. Copying revealed values SHALL be an
explicit action.

#### Scenario: Values open hidden

- **WHEN** the user opens the Values tab
- **THEN** it says the values are hidden and offers a reveal control, with its key shown

#### Scenario: A Secret in the manifest

- **WHEN** a revealed manifest contains `kind: Secret` with `data` and `stringData`
- **THEN** that object shows each key with its value's size, and no value

### Requirement: An earlier revision's values and manifest

From the History tab, the user SHALL be able to open an earlier revision's values and manifest,
with the panel stating which revision it shows.

#### Scenario: Revision 3's values

- **WHEN** the user picks revision 3 in History and chooses View Values
- **THEN** the Values tab shows revision 3's values, still hidden until revealed, and says it shows
  revision 3

### Requirement: Resources the release created

The Resources tab SHALL list every object the deployed revision's manifest declares, with its kind,
name and namespace (the release's namespace when the manifest leaves it out), each as a
`resource-links` reference to that object's own panel, and SHALL mark an object that doesn't exist
in the cluster.

#### Scenario: Following a release's Deployment

- **WHEN** the user selects the release's Deployment `web` in Resources and presses Enter
- **THEN** the Deployment's detail panel opens in the same context

#### Scenario: A declared object is missing

- **WHEN** the manifest declares ConfigMap `web-config` but it doesn't exist
- **THEN** its row is marked missing, and following it opens a panel saying it doesn't exist

### Requirement: Keyboard routes

Every action this capability adds SHALL be reachable from the keyboard as well as the mouse, SHALL
be a registered command with a default binding where it has a key, overridable in `keymap.toml`,
SHALL appear in the command palette while it applies, and the panels SHALL show their keys in a hint
row read from the live keymap.

#### Scenario: Reading a release without the mouse

- **WHEN** the user runs Helm Releases from the palette, moves to a row with the arrow keys, presses
  Enter, and presses the History tab's key
- **THEN** the release's history is shown

### Requirement: Restored with the workspace

A Helm Releases list panel and a release detail panel SHALL be saved with the window's panels and
restored with them, scoped as they were.

#### Scenario: Restart

- **WHEN** the app quits with a release detail panel for `web` in `shop` open and is relaunched
- **THEN** that panel reopens on `web` in `shop` once the context reconnects
