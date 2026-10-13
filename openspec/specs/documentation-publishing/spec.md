# documentation-publishing Specification

## Purpose
Publishes the App repo's documentation site from `docs/site` to GitHub Pages, only from `master`,
with every asset its pages render. Validates the site's structure on every pull request and before
every deploy: a warning-free build, the version manifest against the pages, and internal links.

## Requirements

### Requirement: The documentation site publishes to GitHub Pages from master
The App repository SHALL build the documentation site from `docs/site` and publish it to the
repository's GitHub Pages location when the publishing workflow succeeds on `master`. Pushes to
other branches, including `develop`, SHALL NOT publish.

#### Scenario: A documentation change reaches master
- **WHEN** a push to `master` changes `docs/site` or the publishing workflow
- **THEN** the workflow builds the site and deploys it to GitHub Pages

#### Scenario: A documentation change on develop
- **WHEN** a push to `develop` changes `docs/site`
- **THEN** no Pages deployment runs and the published site is unchanged

#### Scenario: The site build fails
- **WHEN** the Jekyll build fails
- **THEN** the workflow fails without deploying, and the previously published site stays live

### Requirement: The published site carries every asset it renders
The deployment SHALL include the generated HTML and every stylesheet, script, font reference and
image that each published version's pages use.

#### Scenario: A reader opens a published page
- **WHEN** a reader opens a published documentation URL
- **THEN** the page and its styles, scripts and images load from the Pages deployment

### Requirement: Documentation changes are validated before merge and before deploy
The publishing workflow SHALL validate the documentation site on every pull request that changes
it and before every deployment, and a failed validation SHALL stop the deployment. The same
validation SHALL run locally with one documented command.

#### Scenario: A pull request breaks the build
- **WHEN** a pull request changes `docs/site` and the build prints a warning or uses an unknown
  Liquid filter
- **THEN** the workflow fails

#### Scenario: Validation fails on master
- **WHEN** validation fails in a run that would deploy
- **THEN** no deployment runs, and the previously published site stays live

#### Scenario: An author checks the site locally
- **WHEN** an author runs the documented command from `docs/site`
- **THEN** it runs the same build, manifest and link checks as the workflow

### Requirement: Validation checks the version manifest against the pages
Validation SHALL fail when a version's `pages` entry has no source file or built page, when a `nav`
path is not in its version's `pages`, when a page under a version directory is missing from that
version's `pages`, or when a version directory is not in `_data/versions.yml`.

#### Scenario: A page is missing from the manifest
- **WHEN** a page exists under `latest/` but is not in the `latest` version's `pages`
- **THEN** validation fails and names the page

#### Scenario: A nav entry points at an unlisted page
- **WHEN** a version's `nav` path is not one of its `pages`
- **THEN** validation fails and names the entry

### Requirement: Validation checks internal links, not wording
Validation SHALL fail when an internal link or anchor in the built site does not resolve. It SHALL
NOT check external links, and it SHALL NOT assert on page wording.

#### Scenario: A page links to a missing path
- **WHEN** a page links to a path the build doesn't produce
- **THEN** validation fails and names the dead link

#### Scenario: A page's wording changes
- **WHEN** a pull request rewrites a page's text and keeps its links and manifest entries valid
- **THEN** validation passes
