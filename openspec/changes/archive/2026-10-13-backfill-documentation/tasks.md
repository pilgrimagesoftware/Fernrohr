# Tasks

## 1. Navigation and landing page

- [x] 1.1 Add every new page to the `latest` version's `pages` and `nav` in
  `docs/site/_data/versions.yml`, in reading order: install, clusters, windows and panels,
  resources, details, logs and shells, tunnels and forwards, agents, settings, then the reference
  pages. Verify: `bundle exec jekyll build` renders every entry in the sidebar.
- [x] 1.2 Rewrite `docs/site/latest/index.md` to link every guide and reference page, grouped as
  getting started, using Fernrohr, and reference. Verify: no link on the page is a dead path.

## 2. Installation

- [x] 2.1 Write `docs/site/latest/install.md` from `release-packaging` and the README's Running the
  App and Installing on Linux sections. Cover the macOS `.dmg` (Apple silicon) with the quarantine
  workaround (`xattr -d com.apple.quarantine`, Open Anyway, `codesign` for older builds) and the
  Keychain prompt. Cover the Linux `.deb` and `.rpm` with install commands and declared dependencies,
  the AppImage host requirements (`ssh` on `PATH`, a running Secret Service provider) and what fails
  without them, `SHA256SUMS` verification, and where the kubeconfig is read from. Verify: each
  artifact `release-packaging` lists has install steps, and each gotcha has a workaround.

## 3. Guides

- [x] 3.1 Write `clusters.md` from `cluster-connection` and `cluster-picker`. Verify: picker and
  connection states match those specs.
- [x] 3.2 Write `windows-panels.md` from `app-shell`, `application-menu` and `saved-panel-layouts`.
  Verify: window, panel, tab and layout actions match those specs.
- [x] 3.3 Write `resources.md` from `resource-browser`, `resource-group-navigation`,
  `resource-icons`, `namespace-sets` and `events-browser`. Verify: list, navigation and namespace
  behavior match those specs.
- [x] 3.4 Write `details.md` from `object-detail`, `pod-detail` and `resource-links`. Verify: the
  detail views and link-following match those specs.
- [x] 3.5 Write `logs-shells.md` from `pod-logs`, `exec-terminal` and the README Shells section.
  Verify: log follow and scrollback, plus shell keys and exit behavior, match those specs.
- [x] 3.6 Write `tunnels-forwards.md` from `tunnel-config`, `manual-tunnel`,
  `context-tunnel-binding`, `managed-forward`, `port-forward-indicators` and the README Tunnels
  section, using placeholder hosts and contexts only. Verify: tunnel kinds, binding and forward
  indicators match those specs.
- [x] 3.7 Write `agents.md` from `agent-mcp` and the README Agents section, covering setup, what an
  agent can reach, and approving actions and connections. Verify: scope and approval rules match the
  spec.
- [x] 3.8 Write `settings.md` from `typography`, `visual-language`, `desktop-notifications`,
  `about-window` and `report-issue`. Verify: each setting and window described exists and behaves
  as specified.
- [x] 3.9 End every guide with a note naming its source specs. Verify: every spec in
  `openspec/specs/` is named by at least one page.

## 4. Reference

- [x] 4.1 Write `reference/keybindings.md` from `command-system` and every command registered in
  `app/src/command.rs` (title, default binding, `KeyContext`, menu location), plus chords, the
  shortcut timeout and `keymap.toml` overrides from the README Keyboard section. Verify: the list
  matches the registry one-for-one.
- [x] 4.2 Write `reference/mcp-tools.md` from `agent-mcp` and every tool in `app/src/mcp/tools.rs`,
  with parameters, whether it reads, acts after approval or changes the view, and what it returns.
  Verify: the list matches the registry one-for-one. Re-run 3.9 against any spec archived since
  this proposal.

## 5. README and validation

- [x] 5.1 Trim the App `README.md` to a summary, contributor build instructions, and links to the
  published site. Remove the sections now on the site. Verify: no installation, keyboard, shell,
  tunnel or agent content remains duplicated.
- [x] 5.2 Build with `bundle exec jekyll build` from `docs/site`. Verify: no Liquid or Markdown
  errors.
- [x] 5.3 Open every path in `_data/versions.yml` in the generated output. Verify: no navigation
  target is missing, and switching to `v0.9` from a guide lands on the `v0.9` landing page.
- [x] 5.4 Search the new pages for real cluster or context names, IPs, bastions and cloud project
  IDs. Verify: only placeholders appear.

## 6. Process checks

- [x] 6.1 Remove `ui::manual_tunnel::tests::the_manual_tunnel_docs_name_no_infrastructure`, which
  asserted on README wording that moved to the site. Docs tests check the process, never the
  wording. Verify: `cargo test` passes and no `include_str!` reads README or docs content.
- [x] 6.2 Add `docs/site/_scripts/check-manifest.rb`. It checks every version in
  `_data/versions.yml` for missing source or built pages, `nav` paths outside `pages`, unlisted
  pages, and unlisted version directories. Verify: a planted unlisted page and a planted bad `nav`
  path each fail the check.
- [x] 6.3 Add `docs/site/_scripts/check.rb`. It builds with an empty base URL, fails on build
  warnings and unknown Liquid filters, runs the manifest check, and checks internal links with
  html-proofer (external links off). Verify: a planted dead link and a planted unknown filter each
  fail the check.
- [x] 6.4 Run the check in `.github/workflows/docs-pages.yml` on every run, ahead of the Pages build
  and deploy. Verify: the pull request's workflow runs the check, and a failure stops the job before
  upload.
- [x] 6.5 Document `bundle exec ruby _scripts/check.rb` in `docs/site/PUBLISHING.md` and
  `docs/site/AGENTS.md`, and remove the manual pre-merge steps it replaces.
