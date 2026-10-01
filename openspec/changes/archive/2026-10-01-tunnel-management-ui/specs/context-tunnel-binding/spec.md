## MODIFIED Requirements

### Requirement: Bind a context to at most one tunnel

The application SHALL let the user choose, per kube context, either a direct connection or exactly
one tunnel, from that context's entry in the cluster picker and from a "Set tunnel for context"
command. Many contexts MAY choose the same tunnel. The choice SHALL be persisted across restarts,
SHALL be resolved each time a connection to that context starts, and SHALL NOT alter a connection
that is already established.

#### Scenario: Bind and persist

- **WHEN** the user sets context `qa-1`'s tunnel to `qa-bastion` and relaunches
- **THEN** `qa-1` still shows `qa-bastion` as its tunnel, and connecting it routes through `qa-bastion`

#### Scenario: Shared tunnel

- **WHEN** contexts `qa-1` and `qa-2` both use `qa-bastion` and point at the same API server
- **THEN** connecting either context uses the same single tunnel instance

#### Scenario: Unbind

- **WHEN** the user sets `qa-1`'s tunnel to Direct
- **THEN** connecting `qa-1` no longer routes through any tunnel

#### Scenario: Change applies at the next connection

- **WHEN** `qa-1` is connected through `qa-bastion` and the user changes its tunnel to `qa-bastion-ops`
- **THEN** the live connection keeps using `qa-bastion`, and the next connection to `qa-1` uses `qa-bastion-ops`

#### Scenario: Binding visible on the context

- **WHEN** the cluster picker lists contexts
- **THEN** each bound context shows the name of its tunnel, and each unbound context shows it connects directly

## ADDED Requirements

### Requirement: Forward target derived from the context

When a bound context connects, the application SHALL forward through the tunnel to the host and port
of that context's kubeconfig API server URL, using the scheme's default port when the URL omits
one. Forwards SHALL be shared per tunnel and API server host and port: contexts using the same
tunnel and API server share one forward, and contexts using the same tunnel for different API
servers each get their own.

#### Scenario: Target from the kubeconfig server

- **WHEN** a context whose kubeconfig server is `https://10.0.0.10` connects through `qa-bastion`
- **THEN** the tunnel forwards to `10.0.0.10:443` through the bastion

#### Scenario: One bastion, two clusters

- **WHEN** contexts for two different API servers both use `qa-bastion` and both connect
- **THEN** two forwards run through `qa-bastion`, each reaching its own API server

#### Scenario: Unsupported server URL

- **WHEN** a bound context's kubeconfig server URL has no host
- **THEN** the connection fails naming the invalid server URL and no tunnel is started

### Requirement: Stale bindings are surfaced

The application SHALL detect bindings whose context no longer exists in the kubeconfig, SHALL list
them in the Tunnels panel, and SHALL let the user remove them there.

#### Scenario: Context renamed away

- **WHEN** a bound context is renamed or removed in the kubeconfig and the application reloads contexts
- **THEN** the Tunnels panel lists the old context name as a stale binding with its tunnel

#### Scenario: Clear a stale binding

- **WHEN** the user removes a stale binding from the Tunnels panel
- **THEN** it is deleted from `tunnels.toml` and no longer listed
