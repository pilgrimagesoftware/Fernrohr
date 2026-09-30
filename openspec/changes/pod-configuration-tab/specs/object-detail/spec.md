# Spec Delta

## RENAMED Requirements

- FROM: `### Requirement: Secret values are never shown`
- TO: `### Requirement: Secret values are hidden until revealed`

## MODIFIED Requirements

### Requirement: Secret values are hidden until revealed
The object panel SHALL NOT display a Secret value in its YAML view. In its structured view it SHALL
show each key's name and the value's size, with a reveal control per key that shows that one value
until it is hidden, the panel closes, or Hide Secret Values runs. A value SHALL NOT be written to
disk, logged, or placed in a window title, tab name, palette entry or notification.

#### Scenario: Structured view of a Secret
- **WHEN** a Secret's panel shows its structured view
- **THEN** it shows the Secret's type and each key with its size and a reveal control, and no value

#### Scenario: Revealing one value
- **WHEN** the user activates the reveal control on one key
- **THEN** that key's value is shown, and no other

#### Scenario: YAML view of a Secret
- **WHEN** a Secret's panel shows its YAML view
- **THEN** every `data` and `stringData` value, and any last-applied-configuration annotation, is
  replaced with a placeholder giving its size, even while a value is revealed in the structured view

#### Scenario: A revealed value stays out of saved state
- **WHEN** a value is revealed and the window's layout is saved
- **THEN** the saved layout contains no Secret value
