# Spec Delta

## ADDED Requirements

### Requirement: List panel titles use the kind's plural form
A resource-browser panel showing a list of a kind SHALL title its tab and title bar with that
kind's plural display name, not its singular Kubernetes Kind name.

#### Scenario: Pods list panel is titled "Pods"
- **WHEN** a Pods list panel is open
- **THEN** its tab and title bar read "Pods", not "Pod"
