# Tasks

## 1. Site foundation and theme

- [x] 1.1 Inventory the theme sources: accent and palette in `app/src/ui/accent.rs` and
  `app/src/ui/theme.rs`, spacing in `app/src/ui/space.rs`, fonts in `app/src/ui/typography.rs` and
  `app/assets/fonts/`, and the icon in `images/fernrohr-icon.png`. Record the light and dark values.
  Verify: every token in the stylesheet traces to one of those files.
- [x] 1.2 Create `docs/site` with `_config.yml`, `.ruby-version`, `Gemfile`, `Gemfile.lock`,
  `_layouts/default.html`, and `assets/css/site.css` holding the semantic light and dark tokens and
  the spacing scale. Add a reduced-size copy of the icon under `assets/icons/`. Verify:
  `bundle exec jekyll build` completes from `docs/site`.
- [x] 1.3 Render headings, the header, the sidebar and the selectors in Adamina, body text in
  Manrope, and code in Monaco with a monospace fallback. Add the System/Light/Dark appearance
  selector. Verify: in both modes, text meets 4.5:1 and the accent and status tones meet 3:1
  against their surfaces, checked with a contrast tool on the built page.
- [x] 1.4 Seed `latest/index.md` with an overview of Fernrohr. Verify: the built page renders the
  icon, navigation and theme.

## 2. Versioned navigation

- [x] 2.1 Add `_data/versions.yml` with `latest` and `v0.9`, each with `id`, `label`, `path`,
  `pages` and `nav`, plus a `v0.9/index.md` landing page. Verify: the manifest lists both versions
  in display order and the build emits `/latest/` and `/v0.9/`.
- [x] 2.2 Add the root `index.html` that redirects to `/latest/`, and render the sidebar from the
  selected version's `nav` entries. Verify: opening the built root shows the latest documentation.
- [x] 2.3 Add `_includes/version-selector.html` with a `<select>` and `<noscript>` links. Verify: a
  reader can reach `v0.9` both with and without JavaScript.
- [x] 2.4 Map the current page into the selected version, falling back to that version's landing
  page. Verify: switching from a page missing from `v0.9` lands on `/v0.9/` with `v0.9` shown as
  selected.

## 3. Authoring rules and agent instructions

- [x] 3.1 Write `docs/site/AGENTS.md` covering structure, front matter, Markdown, Liquid
  boundaries, `relative_url` links, assets, theme source files, placeholder-only examples, local
  validation, snapshots and publishing. Verify: a new page can be authored from it alone.
- [x] 3.2 Write `docs/site/PUBLISHING.md`: the Pages source setting, publishing from `master`, and
  the release snapshot steps. Verify: it names every path to check after a deploy.
- [x] 3.3 Add a Documentation section to the App `AGENTS.md`: features ship with docs, bug fixes
  update affected docs, docs live in `docs/site/`, they are versioned with latest as the default,
  and the install page covers platform gotchas. Point it at `docs/site/AGENTS.md`. Verify: the
  section states both rules.
- [x] 3.4 Add the same Documentation section to the meta repo `AGENTS.md` (`CLAUDE.md` links to
  it), pointing at `App/docs/site/AGENTS.md`. Verify: `CLAUDE.md` shows it.
- [x] 3.5 Add a `rules:` block to the meta `openspec/config.yaml` with a `tasks` rule: every
  user-facing change includes a task to write or update its `docs/site` pages, and bug fixes update
  affected pages. Leave `context:` untouched so `docs/architecture.md` needs no change. Verify:
  `openspec instructions tasks --change <any active change>` includes the rule.

## 4. GitHub Pages publication

- [x] 4.1 Add `.github/workflows/docs-pages.yml`: build with `bundle exec jekyll build --baseurl`
  from `configure-pages`, upload with `upload-pages-artifact`, deploy with `deploy-pages`.
  Restrict `pages: write` and `id-token: write` to the jobs that need them and use
  `contents: read` everywhere else. Verify: `actionlint` passes on the workflow.
- [x] 4.2 Trigger on pushes to `master` that touch `docs/site/**` or the workflow, plus
  `workflow_dispatch`. Verify: a push to `develop` doesn't start the workflow, and deploy `needs`
  build, so a failed build never deploys.
- [x] 4.3 Set the repository's Pages source to GitHub Actions, then run the workflow by
  `workflow_dispatch` on `master` after merge. Verify: the root, `/latest/` and `/v0.9/` URLs
  serve.

## 5. Validation

- [x] 5.1 Build locally and check internal links across `latest` and `v0.9`. Verify: no broken
  version-selector or navigation target.
- [x] 5.2 Compare the built site with the running app in light and dark mode. Verify: the accent,
  surfaces, fonts and icon match their sources.

## Implementation notes

- Implemented in pilgrimagesoftware/Fernrohr-App#254. Deviations worth a look:
  - 1.1: the base palette isn't defined in `theme.rs`. It is gpui-component's Default Light and
    Default Dark theme, and `app/src/ui/style.rs` derives the surfaces from it and runs the accent
    and status tones through `meet()`. The site applies the same derivation offline. It checks the
    tones against the card surface, the strictest one they sit on, which darkens light-mode green,
    cyan and yellow slightly further than the app does. The values and their sources are in the
    `site.css` header comment.
  - The fonts are served from the bundled OFL TTFs, not Google Fonts, so the deployment carries
    every asset it renders and makes no third-party requests.
  - 4.1: `actionlint` wasn't installed locally. The workflow was parsed with Ruby's YAML loader and
    exercised by the PR's own run. The workflow also builds pull requests, without deploying, so a
    docs change gets a check before it merges. `develop` pushes still never publish.
  - 4.3: the user enabled Pages (`build_type=workflow`) and allowed `master` in the `github-pages`
    environment. The first deploy waits on the next release merge to `master`.
  - 5.2: tokens were compared against their sources and screenshotted in light and dark mode. The
    app itself wasn't run side by side.
