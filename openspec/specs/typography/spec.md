## Purpose

Defines which font families the application renders UI text and monospace/code text in, so
typography is a deliberate choice rather than whatever a library default happens to be.

## Requirements

### Requirement: UI and data text use distinct bundled fonts
The application SHALL render its frame text (panel titles, tabs, section headers, buttons, menus,
the context bar, the status bar and dialogs) in Adamina, and the data inside panels (table cells,
field labels and values, chips and cards) in Manrope. Both SHALL be bundled with the application
rather than depending on them being installed on the host system.

#### Scenario: UI font applied at startup
- **WHEN** the application finishes starting up
- **THEN** panel titles and tabs render in Adamina, and table cells and field values render in
  Manrope

#### Scenario: UI font persists across theme changes
- **WHEN** the user switches between light and dark theme, or the OS appearance changes while
  the theme preference is System
- **THEN** frame text continues to render in Adamina and data text in Manrope

### Requirement: Monospace surfaces render in a real monospace font

Every surface that renders monospaced content (resource YAML detail, pod log output) SHALL
render using Monaco where available, and SHALL fall back to a monospace font family that is
actually present on the host rather than an unstyled default when Monaco is unavailable.

#### Scenario: Monaco applied on a host where it is installed

- **WHEN** the application starts on a host with Monaco installed
- **THEN** YAML detail and log output render in Monaco

#### Scenario: Fallback on a host without Monaco

- **WHEN** the application starts on a host without Monaco installed
- **THEN** YAML detail and log output render in a monospace font family present on that host,
  and text remains legible (not rendered in a proportional font)

### Requirement: Configurable text size
The application SHALL provide a text-size preference that scales UI, data and code text together,
defaulting to the sizes the application uses today. It SHALL be adjustable in Settings and through
Increase Text Size, Decrease Text Size and Reset Text Size commands, apply to every open window
without a restart, stay within a fixed minimum and maximum, and persist in the preference file.

#### Scenario: Increasing text size
- **WHEN** the user runs Increase Text Size
- **THEN** text in every open window grows by one step, and panel spacing scales with it

#### Scenario: Reset to the default
- **WHEN** the user runs Reset Text Size
- **THEN** text returns to the default size

#### Scenario: The size is remembered
- **WHEN** the user changes the text size and relaunches
- **THEN** the application opens at the chosen size

#### Scenario: Bounded
- **WHEN** the text size is at its maximum and the user runs Increase Text Size
- **THEN** nothing changes

### Requirement: Text renders smoothly
Text SHALL be rendered with antialiasing, line heights and weights chosen so that it reads without
visible jaggedness or crowding at the default size on a standard and a high-density display.

#### Scenario: Body text at the default size
- **WHEN** a data table is shown at the default text size on a high-density display
- **THEN** its text renders antialiased, with line spacing that keeps rows from touching
