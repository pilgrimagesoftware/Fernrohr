# Spec Delta

## ADDED Requirements

### Requirement: Object panels show pinned state and a pin/unpin action
The object detail panel SHALL show whether its object is pinned, per `favorites`, and SHALL
offer a pin/unpin action reachable from both mouse and keyboard.

#### Scenario: An unpinned object's panel
- **WHEN** an object's panel is open and the object is not pinned
- **THEN** the panel offers a pin action

#### Scenario: Pinning from the panel
- **WHEN** the user activates the pin action on an unpinned object's panel
- **THEN** the object is pinned and the panel's action becomes unpin
