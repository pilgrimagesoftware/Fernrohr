# documentation-authoring Specification

## Purpose
The rules for writing the documentation site, and the rule that features and bug fixes keep it
current, carried in both repos' agent instructions and the OpenSpec project rules.

## Requirements

### Requirement: Authoring guidance defines the site structure
The App repository SHALL provide `docs/site/AGENTS.md`, which names the documentation root, where
current and snapshotted pages go, the required front matter, the navigation data file, the shared
assets, and the application source files the theme tokens come from.

#### Scenario: An agent adds a page
- **WHEN** an agent creates a documentation page
- **THEN** the guidance tells it the destination directory, the required front matter, and how to
  add the page to navigation

### Requirement: Authoring guidance defines formatting and Jekyll syntax
The guidance SHALL define Markdown conventions, front matter, internal links, code blocks, images,
and where Liquid may be used. It SHALL forbid real cluster or context names, hostnames, IP
addresses, bastions, cloud project IDs and credentials in examples, and require placeholders
instead.

#### Scenario: An agent writes a tunnel example
- **WHEN** an agent documents a tunnel or context configuration
- **THEN** the example uses placeholder names and addresses, not real infrastructure

### Requirement: Authoring guidance defines the publication workflow
The guidance SHALL explain local validation, cutting a release snapshot, pull request expectations,
and the fact that merges to `master` publish the site.

#### Scenario: An agent prepares documentation for a release
- **WHEN** an agent changes documentation intended for a release
- **THEN** the guidance names the local build, the paths to check, and the snapshot step to run
  before the release merge

### Requirement: New features ship with documentation
A change that adds or changes user-facing behavior SHALL add or update the documentation describing
it in the same change, with concise, realistic examples. The OpenSpec project rules SHALL require
every user-facing change's `tasks.md` to include a `docs/site` task.

#### Scenario: A user-facing change is proposed
- **WHEN** an OpenSpec change adds a user-facing feature
- **THEN** its `tasks.md` includes a task to write or update the affected `docs/site` pages

#### Scenario: A feature is implemented without docs
- **WHEN** a pull request adds a user-facing feature and changes no `docs/site` page
- **THEN** the change is incomplete under the agent instructions

### Requirement: Bug fixes update affected documentation
A bug fix that changes behavior described on a documentation page SHALL update that page in the same
change. A fix that shows a page to be wrong SHALL correct the page as well.

#### Scenario: A fix changes documented behavior
- **WHEN** a bug fix changes how a documented feature behaves
- **THEN** the same change updates the page that describes it

### Requirement: Agent instructions carry the documentation rules
The App `AGENTS.md`, the meta repository's `AGENTS.md`, and the `rules:` block of the meta
repository's `openspec/config.yaml` SHALL each state that features ship with documentation and that
bug fixes update affected documentation, and SHALL point at `docs/site/AGENTS.md`.

#### Scenario: An agent starts work in either repository
- **WHEN** an agent reads the repository's agent instructions
- **THEN** it finds the documentation rules and the pointer to the site authoring guide
