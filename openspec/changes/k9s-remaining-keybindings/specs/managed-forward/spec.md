# Spec Delta

## ADDED Requirements

### Requirement: Start a port-forward from a resource-browser row

A resource-browser panel SHALL provide a "port-forward" command on a Pod or Service row that
acquires a Kubernetes-backed managed forward to that resource and surfaces it in the tunnel
management UI, without requiring the user to pre-configure a forward.

#### Scenario: Start a port-forward from a pod row

- **WHEN** the user invokes port-forward on a selected Pod and a target port
- **THEN** a Kubernetes managed forward to that Pod's port is acquired and appears in the tunnel
  management list in its current state

#### Scenario: Multiple ports on the target

- **WHEN** the user invokes port-forward on a resource exposing more than one port
- **THEN** the application asks which port to forward before acquiring the managed forward

#### Scenario: Stopping it from tunnel management releases it

- **WHEN** the user stops a panel-initiated port-forward from the tunnel management UI
- **THEN** the managed forward is released following its normal reference-counted shutdown
