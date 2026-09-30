## Purpose

Defines which font families the application renders UI text and monospace/code text in, so
typography is a deliberate choice rather than whatever a library default happens to be.

## Requirements

### Requirement: UI text renders in Manrope

The application SHALL render its user interface text (labels, buttons, menus, panel titles) in
the Manrope font family, bundled with the application rather than depending on it being
installed on the host system.

#### Scenario: UI font applied at startup

- **WHEN** the application finishes starting up
- **THEN** UI text renders using Manrope

#### Scenario: UI font persists across theme changes

- **WHEN** the user switches between light and dark theme, or the OS appearance changes while
  the theme preference is System
- **THEN** UI text continues to render using Manrope

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
