# Spec Delta

## MODIFIED Requirements

### Requirement: Main window shell provides top-level chrome
The main window SHALL provide consistent top-level chrome with clear structure. The top bar SHALL use the gpui-kit Toolbar component with a defined layout.

#### Scenario: Main window uses gpui-kit Toolbar for top bar
- **WHEN** the main window is rendered
- **THEN** the top bar SHALL be implemented using the gpui-kit Toolbar component

#### Scenario: Toolbar displays expected layout elements
- **WHEN** viewing the main window toolbar
- **THEN** the toolbar SHALL contain, in order: icon, app name, connected contexts list, and an add context button

#### Scenario: Connected contexts remain visible and functional
- **WHEN** contexts are connected
- **THEN** the connected contexts list SHALL remain visible in the toolbar and show the connected contexts as before

#### Scenario: Add context action remains accessible
- **WHEN** the user interacts with the add context button in the toolbar
- **THEN** the existing "add context" behavior/functionality SHALL be preserved and accessible via mouse and keyboard
