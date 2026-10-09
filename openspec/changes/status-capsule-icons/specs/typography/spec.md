# Spec Delta

## MODIFIED Requirements

### Requirement: UI and data text use distinct bundled fonts
The application SHALL render its frame text (panel titles, tabs, section headers, buttons, menus,
the toolbar, the status bar and dialogs) in Adamina, and the data inside panels (table cells,
field labels and values, chips and cards) in Manrope. A status-bar capsule's tunnel name and
elapsed time SHALL render in Manrope. Both SHALL be bundled with the application
rather than depending on them being installed on the host system.

#### Scenario: UI font applied at startup
- **WHEN** the application finishes starting up
- **THEN** panel titles and tabs render in Adamina, and table cells and field values render in
  Manrope

#### Scenario: Status bar capsule fonts
- **WHEN** a status-bar capsule shows a context bound to a tunnel
- **THEN** the context name renders in Adamina and the tunnel name renders in Manrope

#### Scenario: UI font persists across theme changes
- **WHEN** the user switches between light and dark theme, or the OS appearance changes while
  the theme preference is System
- **THEN** frame text continues to render in Adamina and data text in Manrope
