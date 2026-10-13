# documentation-publishing Specification

## Purpose
Publishes the App repo's documentation site from `docs/site` to GitHub Pages, only from `master`,
with every asset its pages render.

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
