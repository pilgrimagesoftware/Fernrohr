## Purpose

Associating a kube context with a tunnel so that connecting the context routes through that tunnel,
gating the connection on tunnel readiness and adjusting the client's address and TLS expectations
accordingly.

## ADDED Requirements

### Requirement: Bind a context to at most one tunnel

The application SHALL let the user bind a kube context to zero or one tunnel, SHALL allow many
contexts to share one tunnel, and SHALL persist bindings across restarts.

#### Scenario: Bind and persist

- **WHEN** the user binds context `qa-1` to tunnel `qa-bastion` and relaunches
- **THEN** `qa-1` is still bound to `qa-bastion`

#### Scenario: Shared tunnel

- **WHEN** contexts `qa-1` and `qa-2` are both bound to `qa-bastion`
- **THEN** connecting either context uses the same single tunnel instance

#### Scenario: Unbind

- **WHEN** the user removes `qa-1`'s binding
- **THEN** connecting `qa-1` no longer routes through any tunnel

### Requirement: Connection gated on tunnel readiness

When a bound context is connected, the application SHALL acquire its tunnel and wait for the tunnel to
be Up before building the Kubernetes client, and SHALL release the tunnel when the last cluster
session using it disconnects.

#### Scenario: Wait for tunnel

- **WHEN** the user connects a context bound to a tunnel that is not yet Up
- **THEN** the connection shows a waiting-for-tunnel state and proceeds once the tunnel reaches Up

#### Scenario: Tunnel fails to start

- **WHEN** the bound tunnel cannot reach Up
- **THEN** the connection fails with the tunnel's failure reason and no client is built

#### Scenario: Tunnel released when unused

- **WHEN** the last connected context using a tunnel disconnects
- **THEN** the tunnel stops

### Requirement: Client address and TLS server name rewrite

For a bound context, the application SHALL build the Kubernetes client against the tunnel's local
loopback address and port, with the TLS server name pinned to the context's original API server host
so that server-certificate validation still succeeds.

#### Scenario: Rewritten client still validates TLS

- **WHEN** a context whose kubeconfig server is `https://api.internal:6443` is connected through a tunnel
- **THEN** the client connects to the tunnel's `127.0.0.1` port and validates the server certificate against `api.internal`

#### Scenario: No insecure fallback

- **WHEN** a bound context is connected
- **THEN** certificate verification is not disabled or skipped as part of the rewrite
