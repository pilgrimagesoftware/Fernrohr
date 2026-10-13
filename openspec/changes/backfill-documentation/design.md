# Design

## Context

See `proposal.md` for the motivation and the `documentation-content` delta for behavior.
`docs-github-pages` provides the Jekyll site, the version manifest with per-version `nav`, the
layout that renders it, and the authoring rules. This change adds the content.

The behavior each page describes is already pinned by the capability specs in `openspec/specs/`.
The pages restate those specs for readers; they are not a second source of truth. Where a page and
a spec disagree, the spec wins and the page is the defect. Several README sections already explain
the same behavior in reader-facing terms, and they are the starting text for the matching pages.

## Goals / Non-Goals

**Goals:**

- One page per user-facing capability area, written as tasks a reader performs, covering every
  capability spec.
- An installation page that gets a first-time user running on macOS or Linux, including the
  platform gotchas that block a first launch.
- Reference pages a reader scans rather than reads: keybindings and commands, and MCP agent tools.
- One home for user documentation: the site, not the README.

**Non-Goals:**

- Generating pages from `openspec/specs/` automatically. The specs are written for implementers and
  change too often to publish verbatim.
- Documenting internal structure (the tokio/GPUI bridge, watcher refcounting, crate layout), or
  anything a person running the packaged app can't observe. That stays in `docs/architecture.md`.
- Backfilling the `v0.8` snapshot beyond its landing page.
- Documenting in-flight changes. A change that is still under `openspec/changes/` documents itself
  through the `docs/site` task that the project rules now require.

## Decisions

### Group capability specs into reader-facing pages

| Page | Capability specs |
|---|---|
| `install` | `release-packaging`, plus the README's running-the-app and Linux sections |
| `clusters` | `cluster-connection`, `cluster-picker` |
| `windows-panels` | `app-shell`, `application-menu`, `saved-panel-layouts` |
| `resources` | `resource-browser`, `resource-group-navigation`, `resource-icons`, `namespace-sets`, `events-browser` |
| `details` | `object-detail`, `pod-detail`, `resource-links` |
| `logs-shells` | `pod-logs`, `exec-terminal` |
| `tunnels-forwards` | `tunnel-config`, `manual-tunnel`, `context-tunnel-binding`, `managed-forward`, `port-forward-indicators` |
| `agents` | `agent-mcp` (setup, scope, approving actions and connections) |
| `settings` | `typography` (text size), `visual-language` (theme and accent), `desktop-notifications`, `about-window`, `report-issue` |
| `reference/keybindings` | `command-system`, with every registered command |
| `reference/mcp-tools` | `agent-mcp`, with every tool |

Any capability spec archived between this proposal and its implementation gets a row too, either
in an existing page or in a new one. Task 4.2 checks this.

One page per spec was rejected. It gives about thirty short pages that duplicate the specs and are
hard to navigate.

### Reference pages come from the code's registries

The keybindings page lists every registered command from `app/src/command.rs`: its title, default
binding, `KeyContext` and menu location. It also covers chorded shortcuts, the shortcut timeout and
`keymap.toml` overrides. The MCP tools page lists every tool registered in `app/src/mcp/tools.rs`, with its
parameters, whether it is read-only or needs approval in the app, and what it returns. Both pages
are checked against the registries rather than written from memory. Each one names its source file
so the check can be repeated.

### The install page owns the platform gotchas

- macOS: Apple silicon `.dmg` only. The app is ad-hoc signed but not notarized, so a downloaded
  copy is quarantined and reported as damaged or from an unidentified developer. The fixes are
  `xattr -d com.apple.quarantine` or Privacy & Security -> Open Anyway, plus `codesign` for builds
  that predate signing. The page also covers the Keychain prompt on first access to stored
  secrets.
- Linux: `.deb`, `.rpm` and AppImage for x86_64 and aarch64, with install commands. The packages
  depend on the SSH client and `libsecret` and recommend a Secret Service provider. The AppImage
  needs `ssh` on `PATH` and a running Secret Service provider on the host. The page also covers
  what happens when no provider is running.
- Both: verifying downloads with `SHA256SUMS`, and where the kubeconfig is read from.

### Move README user docs into the site

The README sections on running the app, installing on Linux, the keyboard, shells, tunnels and
agents move into the matching pages, with their content preserved. The README keeps a one-paragraph
summary, the build instructions for contributors, and a link to the published site. This gives the
docs one home, the same choice Knot-App made for its loose documents.

### Every page names its source specs

Each guide closes with a short note naming the capability specs it summarizes. A spec change then
points at the page to update, and the `docs/site` task rule has a concrete target.

## Risks / Trade-offs

- [Pages drift from the specs as features land] -> Each page names its specs, and the project rules
  require a `docs/site` task in every user-facing change.
- [Navigation data and the page tree disagree] -> Validation opens every path listed in
  `versions.yml` in the built output.
- [Reference pages go stale as commands and tools are added] -> They name their source registries,
  and the authoring rules make updating them part of adding a command or tool.
- [Examples leak real infrastructure] -> Tunnel, context and agent examples use placeholders only,
  as `docs/site/AGENTS.md` requires.

## Migration Plan

1. Add the pages under `docs/site/latest/`, and their `pages` and `nav` entries in
   `_data/versions.yml`.
2. Rewrite `latest/index.md` to link every page.
3. Move the README sections into the site and trim the README.
4. Build locally and open every path in `versions.yml`.
5. Roll back by reverting the change. The site is a static tree.
