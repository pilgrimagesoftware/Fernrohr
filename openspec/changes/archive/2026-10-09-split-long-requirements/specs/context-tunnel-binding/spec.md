# Spec Delta

## MODIFIED Requirements

### Requirement: Forward target derived from the context

When a context bound to an SSH tunnel connects, the application SHALL forward through the tunnel to
the host and port of that context's kubeconfig API server URL, using the scheme's default port when
the URL omits one. SSH forwards SHALL be shared per tunnel and API server host and port: contexts
using the same tunnel and API server share one forward, and contexts using the same tunnel for
different API servers each get their own.

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

### Requirement: Proxy-mode routing

For a context bound to a proxy-mode command tunnel, the application SHALL build the Kubernetes
client against the context's own kubeconfig API server URL and SHALL send all of that client's API
server traffic through the HTTP proxy at `http://127.0.0.1:<local port>`. This includes watches, log
streams, exec sessions, port-forwards and requests to the API server's service proxy.

#### Scenario: Cluster reached through the proxy

- **WHEN** context `qa-1`, whose kubeconfig server is `https://10.0.0.10`, connects through the proxy-mode command tunnel `qa-iap` running on local port 8888
- **THEN** the client's API server connections go through the HTTP proxy at `127.0.0.1:8888` to `10.0.0.10:443`, and the server certificate is validated as for a direct connection

#### Scenario: Proxy scoped to the bound context

- **WHEN** `qa-1` is connected through `qa-iap` and context `dev` with no tunnel is also connected
- **THEN** `dev`'s client connects directly, without using the proxy

#### Scenario: Proxy refuses the connection

- **WHEN** `qa-iap` is Up but the proxy behind it refuses to connect to `qa-1`'s API server
- **THEN** the connection fails with an error naming the proxy failure, and no insecure fallback or direct connection is attempted

## ADDED Requirements

### Requirement: A command tunnel is shared by all its contexts

A command tunnel SHALL NOT derive a target from the context. All contexts bound to the same command
tunnel SHALL share one running command, whatever their API servers.

#### Scenario: Two clusters, one command

- **WHEN** contexts for two different API servers are both bound to one command tunnel and both
  connect
- **THEN** exactly one instance of that tunnel's command is running and both connections use it

### Requirement: A manual tunnel connects its contexts directly

A manual tunnel SHALL NOT derive a target or rewrite the client address; once confirmed, its
contexts SHALL connect directly to their API servers. All contexts bound to the same manual tunnel
SHALL share one confirmation, whatever their API servers.

#### Scenario: Two clusters, one confirmation

- **WHEN** contexts for two different API servers are both bound to one manual tunnel and both
  connect
- **THEN** the user confirms the tunnel once, and each connection reaches its own API server
  directly without an address rewrite

### Requirement: Proxied TLS is end to end

For a context bound to a proxy-mode command tunnel, TLS to the API server SHALL be negotiated end to
end through the proxy and validated against the API server's own host name.

#### Scenario: Certificate validated through the proxy

- **WHEN** a context connects through a proxy-mode command tunnel
- **THEN** the API server's certificate is validated against its own host name, as for a direct
  connection

### Requirement: A tunnel's proxy applies only to its context

A proxy-mode command tunnel's proxy SHALL apply only to the bound context's client, never to the
application's process environment or to other contexts.

#### Scenario: An unbound context connects directly

- **WHEN** one context is connected through a proxy-mode command tunnel and another context with no
  tunnel is also connected
- **THEN** the second context's client connects directly, without using the proxy
