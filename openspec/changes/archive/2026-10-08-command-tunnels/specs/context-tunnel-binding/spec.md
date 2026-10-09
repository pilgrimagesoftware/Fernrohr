# Spec Delta

## MODIFIED Requirements

### Requirement: Client address and TLS server name rewrite

For a context bound to an SSH tunnel or to a forward-mode command tunnel, the application SHALL
build the Kubernetes client against the tunnel's local loopback address and port, with the TLS
server name pinned to the context's original API server host so that server-certificate validation
still succeeds. For a context bound to a proxy-mode command tunnel, the application SHALL NOT
rewrite the client's address or TLS server name.

#### Scenario: Rewritten client still validates TLS

- **WHEN** a context whose kubeconfig server is `https://api.internal:6443` is connected through an SSH tunnel
- **THEN** the client connects to the tunnel's `127.0.0.1` port and validates the server certificate against `api.internal`

#### Scenario: Forward-mode command tunnel is rewritten

- **WHEN** a context is connected through a forward-mode command tunnel
- **THEN** the client connects to the tunnel's `127.0.0.1` port and validates the server certificate against the original API server host

#### Scenario: No insecure fallback

- **WHEN** a bound context is connected
- **THEN** certificate verification is not disabled or skipped as part of the rewrite

### Requirement: Forward target derived from the context

When a context bound to an SSH tunnel connects, the application SHALL forward through the tunnel to
the host and port of that context's kubeconfig API server URL, using the scheme's default port when
the URL omits one. SSH forwards SHALL be shared per tunnel and API server host and port: contexts
using the same tunnel and API server share one forward, and contexts using the same tunnel for
different API servers each get their own. A command tunnel SHALL NOT derive a target from the
context. All contexts bound to the same command tunnel SHALL share one running command, whatever
their API servers.

#### Scenario: Target from the kubeconfig server

- **WHEN** a context whose kubeconfig server is `https://10.0.0.10` connects through `qa-bastion`
- **THEN** the tunnel forwards to `10.0.0.10:443` through the bastion

#### Scenario: One bastion, two clusters

- **WHEN** contexts for two different API servers both use `qa-bastion` and both connect
- **THEN** two forwards run through `qa-bastion`, each reaching its own API server

#### Scenario: One command tunnel, two clusters

- **WHEN** contexts for two different API servers are both bound to the command tunnel `qa-iap` and both connect
- **THEN** exactly one `qa-iap` command is running and both connections use it

#### Scenario: Unsupported server URL

- **WHEN** a context bound to an SSH tunnel has a kubeconfig server URL with no host
- **THEN** the connection fails naming the invalid server URL and no tunnel is started

## ADDED Requirements

### Requirement: Proxy-mode routing

For a context bound to a proxy-mode command tunnel, the application SHALL build the Kubernetes
client against the context's own kubeconfig API server URL and SHALL send all of that client's API
server traffic through the HTTP proxy at `http://127.0.0.1:<local port>`. This includes watches,
log streams, exec sessions, port-forwards and requests to the API server's service proxy. TLS to
the API server SHALL be negotiated end to end through the proxy and validated against the API
server's own host name. The proxy SHALL apply only to that context's client, never to the
application's process environment or to other contexts.

#### Scenario: Cluster reached through the proxy

- **WHEN** context `qa-1`, whose kubeconfig server is `https://10.0.0.10`, connects through the proxy-mode command tunnel `qa-iap` running on local port 8888
- **THEN** the client's API server connections go through the HTTP proxy at `127.0.0.1:8888` to `10.0.0.10:443`, and the server certificate is validated as for a direct connection

#### Scenario: Proxy scoped to the bound context

- **WHEN** `qa-1` is connected through `qa-iap` and context `dev` with no tunnel is also connected
- **THEN** `dev`'s client connects directly, without using the proxy

#### Scenario: Proxy refuses the connection

- **WHEN** `qa-iap` is Up but the proxy behind it refuses to connect to `qa-1`'s API server
- **THEN** the connection fails with an error naming the proxy failure, and no insecure fallback or direct connection is attempted
