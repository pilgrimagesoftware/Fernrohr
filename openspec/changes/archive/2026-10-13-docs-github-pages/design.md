# Design

## Context

See `proposal.md` for the motivation and the three spec deltas for behavior. The App repo has no
documentation-site capability. Its user-facing docs are sections of `README.md`. The site has to
be a static GitHub Pages deployment with its source in `docs/site`, and it has to serve more than
one release's documentation without a server.

The Knot-App repo already ships this setup (its archived `docs-github-pages` change). This change
follows the same shape and adapts it to Fernrohr's branches, release line and visual identity.

## Goals / Non-Goals

**Goals:**

- Build `docs/site` with Jekyll in GitHub Actions and deploy the output with the GitHub Pages
  artifact and deploy actions.
- Keep a `latest/` tree and one directory per snapshotted release, with a data file that declares
  the supported versions, their labels and their navigation.
- Send the site root to `latest`, and render a version selector from the data file.
- Take colors, fonts, spacing and the icon from Fernrohr's own sources, found by an inventory at
  implementation time.
- Put the authoring rules beside the site source. Make keeping the docs current a rule that
  agents and OpenSpec proposals pick up automatically.

**Non-Goals:**

- Serving documentation dynamically, or adding a CMS.
- Generating historical documentation from Git tags.
- A page-for-page match between versions when an older release lacks a newer page.
- Preview sites for pull requests.
- Writing the capability pages. `backfill-documentation` (Fernrohr-App#240) does that. This change
  seeds only an overview and the `v0.9` landing page.

## Decisions

### Publish from `master` through a repository-managed Pages workflow

`.github/workflows/docs-pages.yml` builds with `bundle exec jekyll build --baseurl <pages base
path>` and deploys with `actions/upload-pages-artifact` and `actions/deploy-pages`. It has
`pages: write` and `id-token: write` on the jobs that need them and `contents: read` everywhere
else. It triggers on pushes to `master` that touch `docs/site/**` or the workflow itself, plus
`workflow_dispatch`.

`master` rather than `develop`: `develop` is the default branch, and work on it is ahead of any
release. Publishing from `master` keeps `latest` aligned with what a user can download. The cost
is that documentation merged to `develop` appears only after the next release merge, which is
the intended behavior.

GitHub's branch-source Pages build was considered. It gives less control over the Jekyll toolchain
and can't gate deployment on a successful build, so it was rejected.

### Versions are static source trees with a manifest

`docs/site/_data/versions.yml` lists the versions in display order: `latest` first, then `v0.9`.
Each has an `id`, `label`, `path`, `pages` and `nav`. The build emits `/latest/...` and
`/v0.9/...`, and the root page redirects to `/latest/`. When a release is cut, its documentation
is snapshotted into a new `vX.Y/` directory and added to the manifest. Historical snapshots are
edited only to correct that release's documentation.

`v0.9` is the first snapshot because it is the latest release when the site launches. It starts as a landing page and gains pages only if someone backfills them.

Generating snapshots from Git tags at build time was rejected. It means checking out old trees in
the workflow and makes historical edits impossible to review.

### Version selector with a page-or-landing fallback

The selector is an include rendered on every page from the manifest. It maps the current page into
the chosen version when that version lists the page, and otherwise goes to that version's landing
page. A `<noscript>` block renders plain links so the selector works without JavaScript.

A versioning plugin was considered. A handful of static snapshots doesn't need one.

### Theme tokens come from the application, in one stylesheet layer

`assets/css/site.css` defines semantic custom properties for accent, accent hover, surface,
raised surface, text, muted text, border, and good/warning/bad/neutral tones, in both light and
dark mode. It also defines the spacing scale. The values come from the inventory in task 1.1 of:

- `app/src/ui/accent.rs` and `app/src/ui/theme.rs` (the accent and the theme palette),
- `app/src/ui/space.rs` (the spacing scale),
- `app/src/ui/typography.rs` and `app/assets/fonts/` (Adamina for chrome, Manrope for content,
  Monaco with a monospace fallback for code),
- `images/fernrohr-icon.png` (the application icon, copied to a reduced-size asset).

The site follows the app's split: headings, the header, the sidebar and the selectors are in
Adamina, which is the frame font. Body text is in Manrope, the data font. Code is in Monaco and
falls back to a generic monospace font. The `visual-language` contrast floor applies here as well:
4.5:1 for text and 3:1 for the accent and status tones, in both modes. An appearance selector
offers System, Light and Dark.

The site uses the theme's blue accent. The platform's system accent can't be read from a static
site. A separate documentation theme was rejected because it would drift from the application.

### Documentation is part of done, enforced in three places

The `documentation-authoring` delta requires that a new user-facing feature ships its docs and that
a bug fix updates any page describing the behavior it changes. A rule in one file is easy to miss,
so it lives where each actor reads instructions:

- the App `AGENTS.md`, which agents implementing code read, gets a Documentation section;
- the meta `AGENTS.md` (`CLAUDE.md` is a symlink to it), which agents working on proposals read,
  gets a Documentation section;
- `openspec/config.yaml` gets `rules:` with a `tasks` rule. OpenSpec injects it into every
  proposal, so each user-facing change's `tasks.md` carries a `docs/site` task without anyone
  having to remember it.

The `context:` block is not edited, so `docs/architecture.md` stays in sync as it is.

### Authoring rules live under the site root

`docs/site/AGENTS.md` covers directory roles, front matter (`title`, `version`, permalink),
Markdown conventions, where Liquid is allowed, links through `relative_url`, assets, the theme
source files above, local validation, snapshots and publishing. It also repeats the public-repo
rule: no real cluster or context names, hosts, IPs, bastions or cloud project IDs in examples. Use
placeholders.

## Risks / Trade-offs

- [Historical snapshots drift from their release] -> Snapshots are edited only to correct that
  release's documentation, and `docs/site/AGENTS.md` says so.
- [The application theme changes] -> Tokens live in one stylesheet layer, and the source files are
  listed in `docs/site/AGENTS.md`. A theme change points at that layer.
- [A version lacks the current page] -> The selector falls back to that version's landing page.
- [Pages isn't enabled for the repo] -> `docs/site/PUBLISHING.md` documents the setting, and the
  deploy job fails visibly until it is set.
- [Local and CI builds differ] -> Pin Ruby (`.ruby-version`) and the bundle (`Gemfile.lock`), and
  use the same `bundle exec jekyll build` in both.
- [Docs merged to `develop` lag the published site] -> Intended. `latest` describes the latest
  release, and the next release merge publishes the pending pages.

## Migration Plan

1. Add the Jekyll source tree, theme layer, layouts, includes, version manifest, `latest` seed and
   `v0.9` snapshot.
2. Add `docs/site/AGENTS.md`, `docs/site/PUBLISHING.md`, and the Documentation rules in both
   `AGENTS.md` files and `openspec/config.yaml`.
3. Add the Pages workflow and set the repository's Pages source to GitHub Actions.
4. After the next merge to `master`, verify the root opens `latest`, `v0.9` can be selected, and a
   failing build leaves the live deployment intact.
5. Roll back by reverting the publishing commit, or by re-running the workflow from the last good
   commit.
