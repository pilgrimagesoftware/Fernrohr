# Spec Delta

## ADDED Requirements

### Requirement: Container cards expand to show extended detail
Each container card in the pod detail panel SHALL be expandable to show environment variables,
volume mounts, probes, command/args, and security context, beyond the summary card's fixed
fields.

#### Scenario: Expanding a container card
- **WHEN** the user expands a container's card
- **THEN** it shows environment variables, volume mounts, configured probes, command/args, and
  security context, in addition to the summary fields already shown

### Requirement: Secret-sourced environment values are never resolved
The panel SHALL show a Secret- or ConfigMap-sourced environment variable as its reference (source
kind, name, key), and SHALL NOT display the resolved value.

#### Scenario: A Secret-sourced environment variable
- **WHEN** an expanded container has an environment variable sourced from a Secret
- **THEN** the panel shows which Secret and key it comes from, not the Secret's decoded value
