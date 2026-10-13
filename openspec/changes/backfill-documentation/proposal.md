# Proposal

## Why

`docs-github-pages` gives Fernrohr a versioned documentation site, but its `latest/` tree carries
only an overview. Every capability in `openspec/specs/` has no page on the site. Some of them
already have reader-facing text, in the App `README.md` sections on running the app, Linux
packages, the keyboard, shells, tunnels and agents. That text describes no particular release and
isn't reachable from the site. A reader can't learn from the site what Fernrohr does, how to
install it, or which keys and agent tools it offers.

Tracks pilgrimagesoftware/Fernrohr-App#240. Depends on `docs-github-pages`
(pilgrimagesoftware/Fernrohr-App#239).

## What Changes

- Add guide pages under `docs/site/latest/`, one per user-facing capability area, grouped by the
  task a reader performs. Together they cover every capability spec in `openspec/specs/`.
- Add an installation page covering the macOS `.dmg` with its Gatekeeper quarantine workaround,
  the Linux `.deb`, `.rpm` and AppImage with their runtime dependencies and host requirements,
  and checksum verification. Each platform gets its gotchas.
- Add reference pages for the keybindings and command list and for the MCP agent tools.
- Move the README's user documentation into those pages, and trim the README to a short summary
  that links to the site.
- List every new page in the `latest` version's `pages` and `nav` entries in
  `_data/versions.yml`, which the sidebar already renders from, and link every page from the landing
  page.

## Capabilities

### New Capabilities

- `documentation-content`: which user-facing pages the published site carries, what each page has
  to explain, how pages are addressed and linked, and how navigation comes from version data.

### Modified Capabilities

None.

## Impact

- App repo: `docs/site/latest/` gains guide and reference pages. `docs/site/_data/versions.yml`
  gains `nav` entries for `latest`. `docs/site/latest/index.md` links every page. `README.md`
  loses the sections that moved to the site and links to it instead.
- No application code changes. The pages restate behavior already pinned by `openspec/specs/`.
