# Proposal

## Why

Fernrohr has no documentation site. What user documentation exists lives in the App `README.md`:
installation, keyboard, shells, tunnels and agents. A reader can't tell which release it describes,
and nothing requires a new feature or a bug fix to update it. Users need a public reference that
matches the release they installed. Contributors and agents need one set of authoring rules, plus a
standing requirement that docs move with the code.

Tracks pilgrimagesoftware/Fernrohr-App#239.

## What Changes

- Add a Jekyll documentation site rooted at `docs/site` in the App repo. GitHub Actions publishes it
  to GitHub Pages on pushes to `master`, the release branch. `develop` stays the default and
  integration branch and doesn't publish.
- Version the documentation: `latest` is the default, and a `v0.8` release snapshot can be picked
  from a version selector.
- Add authoring rules for agents (`docs/site/AGENTS.md`) covering structure, front matter, Markdown,
  Liquid, assets, snapshots and publishing.
- Make documentation part of done. A new user-facing feature ships with its docs, and a bug fix that
  changes documented behavior updates the affected pages. Three places carry this rule: the App
  `AGENTS.md`, the meta repo's `AGENTS.md`, and a `rules:` entry in `openspec/config.yaml` that has
  every user-facing change's `tasks.md` include a `docs/site` task.
- Base the site shell on Fernrohr's visual identity: the accent and surface layers from
  `visual-language`, the Adamina/Manrope/Monaco split from `typography`, and the application icon.

## Capabilities

### New Capabilities

- `documentation-publishing`: builds the versioned Jekyll site from `docs/site` and publishes it to
  GitHub Pages from `master`.
- `documentation-authoring`: the site structure, the authoring rules agents follow, and the rule
  that features and bug fixes keep the docs current.
- `documentation-versioning`: `latest` is shown by default, and readers can select a supported
  earlier release.

### Modified Capabilities

None.

## Impact

- App repo: new `docs/site/` (Jekyll config, Gemfile and lockfile, layouts, includes, stylesheet,
  icon asset, `latest/` and `v0.8/` trees, `AGENTS.md`, `PUBLISHING.md`) and
  `.github/workflows/docs-pages.yml`. `AGENTS.md` gains a Documentation section.
- Meta repo: `AGENTS.md` gains a Documentation section. `openspec/config.yaml` gains a `rules:`
  block. `context:` and `docs/architecture.md` are not touched.
- Repository settings: GitHub Pages must be set to deploy from GitHub Actions.
- No application code changes.
