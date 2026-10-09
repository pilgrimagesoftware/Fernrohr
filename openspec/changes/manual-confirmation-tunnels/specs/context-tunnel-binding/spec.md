# Spec Delta

## MODIFIED Requirements

### Requirement: Forward target derived from the context

When a context bound to an SSH tunnel connects, the application SHALL forward through the tunnel to
the host and port of that context's kubeconfig API server URL, using the scheme's default port when
the URL omits one. SSH forwards SHALL be shared per tunnel and API server host and port: contexts
using the same tunnel and API server share one forward, and contexts using the same tunnel for
different API servers each get their own. A command tunnel SHALL NOT derive a target from the
context. All contexts bound to the same command tunnel SHALL share one running command, whatever
their API servers. A manual tunnel SHALL NOT derive a target or rewrite the client
address; once confirmed, its contexts SHALL connect directly to their API servers. All contexts
bound to the same manual tunnel SHALL share one confirmation, whatever their API servers.

#### Scenario: Target from the kubeconfig server

- **WHEN** a context whose kubeconfig server is `https://10.0.0.10` connects through `qa-bastion`
- **THEN** the tunnel forwards to `10.0.0.10:443` through the bastion

#### Scenario: One bastion, two clusters

- **WHEN** contexts for two different API servers both use `qa-bastion` and both connect
- **THEN** two forwards run through `qa-bastion`, each reaching its own API server

#### Scenario: One command tunnel, two clusters

- **WHEN** contexts for two different API servers are both bound to the command tunnel `qa-iap` and both connect
- **THEN** exactly one `qa-iap` command is running and both connections use it

#### Scenario: One manual tunnel, two clusters

- **WHEN** contexts for two different API servers are both bound to the manual tunnel `corp-vpn` and both connect
- **THEN** the user confirms `corp-vpn` once, and each connection reaches its own API server directly without an address rewrite

#### Scenario: Unsupported server URL

- **WHEN** a context bound to an SSH tunnel has a kubeconfig server URL with no host
- **THEN** the connection fails naming the invalid server URL and no tunnel is started
