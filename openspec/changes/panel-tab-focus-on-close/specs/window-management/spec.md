# Spec Delta

## MODIFIED Requirements

### Requirement: Window layout supports docked panels with tabs
The window SHALL support docked panels organized in tabs. Tab operations (close, activate/select) SHALL maintain proper keyboard focus for continued keyboard navigation.

#### Scenario: Closing focused panel tab transfers focus appropriately
- **WHEN** the user closes a panel tab that currently has keyboard focus
- **THEN** keyboard focus SHALL transfer to an appropriate target (such as the adjacent remaining tab, the dock/panel container, or another sensible fallback) and SHALL NOT become lost

#### Scenario: Clicking panel tab after close gives it focus
- **WHEN** a panel tab was closed causing focus loss and the user subsequently clicks another panel tab
- **THEN** that clicked tab SHALL receive keyboard focus and keyboard input SHALL work immediately
