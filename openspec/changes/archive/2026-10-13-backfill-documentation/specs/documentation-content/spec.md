# Spec Delta

## ADDED Requirements

### Requirement: The site documents every user-facing capability
The published site SHALL provide a page for each user-facing capability area of the application,
grouped by the task a reader performs, so that every capability spec in `openspec/specs/` is
covered by at least one page. Pages SHALL describe what a person running the packaged application
can observe and do, through both the keyboard and the mouse, not internal structure.

#### Scenario: A reader opens a capability guide
- **WHEN** a reader opens a guide page
- **THEN** it explains the tasks the capability supports, the keyboard and mouse routes to them, and
  the states the reader can observe

#### Scenario: A capability has no page
- **WHEN** a capability spec in `openspec/specs/` is not covered by any page
- **THEN** the site is incomplete, and the omission is a documentation defect

### Requirement: The site carries an installation page with platform gotchas
The site SHALL provide an installation page that covers every release artifact `release-packaging`
publishes. For macOS, it SHALL explain the Gatekeeper quarantine on the ad-hoc-signed app and how to
clear it. For the Linux `.deb` and `.rpm`, it SHALL give install commands and the runtime
dependencies the packages declare. For the AppImage, it SHALL list its host requirements. It SHALL
explain how to verify a download against `SHA256SUMS`.

#### Scenario: A macOS user is told the app is damaged
- **WHEN** a reader's downloaded copy is blocked by Gatekeeper
- **THEN** the installation page gives the command and the System Settings route that let it open

#### Scenario: A Linux user runs the AppImage without a Secret Service provider
- **WHEN** a reader runs the AppImage on a host with no Secret Service provider running
- **THEN** the installation page names the requirement and the providers that satisfy it

### Requirement: The site carries keybinding and agent-tool references
The site SHALL provide a keybinding reference listing every registered command with its title,
default binding and the context it applies in, and explaining chorded shortcuts and `keymap.toml`
overrides. It SHALL provide an MCP agent tools reference listing every tool the application's MCP
server offers, with its parameters and whether it reads, acts after in-app approval, or changes
what the app shows.

#### Scenario: A reader looks up a shortcut
- **WHEN** a reader needs the key for a command
- **THEN** the keybinding reference lists it with its default binding and context

#### Scenario: A reader sets up an agent
- **WHEN** a reader wants to know what an agent can do through Fernrohr
- **THEN** the MCP tools reference lists each tool and whether it needs approval

### Requirement: Pages are addressed by stable versioned permalinks
Each page SHALL declare a `title`, a `version`, and a permalink under its version path, so it is
reachable at a stable URL and belongs to exactly one documentation version.

#### Scenario: A reader follows a link to a guide
- **WHEN** a reader follows a link to a guide or reference page
- **THEN** the page resolves under its version path and the selector shows that version

### Requirement: Navigation comes from version data
The sidebar for the selected version SHALL be rendered from that version's `nav` entries in
`_data/versions.yml`, and the layout SHALL NOT hard-code page links.

#### Scenario: A page is added
- **WHEN** a page is added and listed in a version's `nav` entries
- **THEN** the sidebar links to it without a layout change

#### Scenario: An older snapshot lacks a page
- **WHEN** a reader selects a version whose `nav` doesn't list a page
- **THEN** that version's sidebar omits the link rather than pointing at a missing path

### Requirement: The site is the single home for user documentation
User documentation SHALL live on the site. The App `README.md` SHALL link to the site instead of
duplicating installation, keyboard, shell, tunnel or agent documentation.

#### Scenario: A reader starts from the README
- **WHEN** a reader looks for installation or usage help in the README
- **THEN** it points them at the corresponding site page

### Requirement: Pages name their source contracts
Each guide and reference page SHALL name the capability specs or source registries it summarizes,
so a change to one of them points at the page to update.

#### Scenario: A spec changes
- **WHEN** a capability spec changes
- **THEN** the page naming that spec identifies itself as the place to update
