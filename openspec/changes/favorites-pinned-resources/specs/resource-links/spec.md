# Spec Delta

## ADDED Requirements

### Requirement: A link shows pinned state and can be pinned without opening its target
A `resource-links` reference SHALL show whether the object it points to is pinned, per
`favorites`, and SHALL offer a pin/unpin action on the reference itself, so a user can pin an
object they haven't navigated to yet.

#### Scenario: Pinning through a link
- **WHEN** a detail view shows a link to an unpinned object and the user activates that link's
  pin action
- **THEN** the referenced object is pinned without that object's own panel opening

#### Scenario: A link to a pinned object shows it
- **WHEN** a detail view shows a link to an object that is already pinned
- **THEN** the link shows the pinned indicator
