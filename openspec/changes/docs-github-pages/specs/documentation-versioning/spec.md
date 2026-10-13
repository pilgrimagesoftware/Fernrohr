# Spec Delta

## ADDED Requirements

### Requirement: Latest documentation is the default entry point
The site SHALL show the `latest` documentation when a reader opens the site root.

#### Scenario: A reader opens the site root
- **WHEN** a reader opens the documentation site's root URL
- **THEN** the site shows the latest documentation

### Requirement: A reader can select a supported version
Every page SHALL show a version selector listing `latest` and each snapshotted release in the
version manifest, starting with `v0.8`. The selector SHALL work without JavaScript through plain
links.

#### Scenario: A reader selects an earlier release
- **WHEN** a reader picks `v0.8` in the version selector
- **THEN** the site opens the `v0.8` documentation

#### Scenario: JavaScript is disabled
- **WHEN** a reader with JavaScript disabled views a page
- **THEN** a link to each supported version is shown

### Requirement: Pages identify their version and keep it when switching
The site SHALL show which version is being viewed. Switching versions SHALL open the same page in
the selected version when that version has it, and that version's landing page otherwise.

#### Scenario: The page exists in both versions
- **WHEN** a reader switches version on a page that both versions carry
- **THEN** the corresponding page opens and the selector shows the selected version

#### Scenario: The page is missing from the selected version
- **WHEN** a reader switches to a version that lacks the current page
- **THEN** that version's landing page opens and the selector shows the selected version

### Requirement: The site follows Fernrohr's visual identity
The site SHALL take its accent, surfaces, status tones, spacing scale and fonts from the
application's theme: Adamina for headings and navigation, Manrope for body text, Monaco with a
monospace fallback for code. Its header SHALL show the application icon. It SHALL offer System,
Light and Dark appearance, and in both modes it SHALL meet the application's contrast floor: 4.5:1
for text, and 3:1 for the accent and status tones against their backgrounds.

#### Scenario: A reader views site navigation
- **WHEN** a reader views any documentation page
- **THEN** the header, sidebar and version selector use the application's accent, fonts and icon

#### Scenario: Dark appearance
- **WHEN** the reader selects Dark, or selects System while the OS is in dark mode
- **THEN** the site switches to the application's dark surfaces and still meets the contrast floor
