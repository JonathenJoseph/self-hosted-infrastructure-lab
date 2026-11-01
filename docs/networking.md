# Networking and Routing

**Snapshot date:** 1 November 2026

## Historical Docker network membership details omitted## Caddy routing

The main Caddyfile imports all `*.caddy` snippets from the sites directory. Nine site snippets were observed. Local hostnames are intentionally redacted in this repository.

| Site snippet | Configured upstream | Routing style | Observation |
|---|---|---|---|
| [REVERSE-PROXY ROUTE] | [UPSTREAM DETAILS REDACTED] | [LOCAL NETWORK/PORT DETAILS OMITTED] | [HISTORICAL NOTE OMITTED] |
| [REVERSE-PROXY ROUTE] | [UPSTREAM DETAILS REDACTED] | [LOCAL NETWORK/PORT DETAILS OMITTED] | [HISTORICAL NOTE OMITTED] |
| [REVERSE-PROXY ROUTE] | [UPSTREAM DETAILS REDACTED] | [LOCAL NETWORK/PORT DETAILS OMITTED] | [HISTORICAL NOTE OMITTED] |
| [REVERSE-PROXY ROUTE] | [UPSTREAM DETAILS REDACTED] | [LOCAL NETWORK/PORT DETAILS OMITTED] | [HISTORICAL NOTE OMITTED] |
| [REVERSE-PROXY ROUTE] | [UPSTREAM DETAILS REDACTED] | [LOCAL NETWORK/PORT DETAILS OMITTED] | [HISTORICAL NOTE OMITTED] |
| [REVERSE-PROXY ROUTE] | [UPSTREAM DETAILS REDACTED] | [LOCAL NETWORK/PORT DETAILS OMITTED] | [HISTORICAL NOTE OMITTED] |
| [REVERSE-PROXY ROUTE] | [UPSTREAM DETAILS REDACTED] | [LOCAL NETWORK/PORT DETAILS OMITTED] | [HISTORICAL NOTE OMITTED] |
| [REVERSE-PROXY ROUTE] | [UPSTREAM DETAILS REDACTED] | [LOCAL NETWORK/PORT DETAILS OMITTED] | [HISTORICAL NOTE OMITTED] |
| [REVERSE-PROXY ROUTE] | [UPSTREAM DETAILS REDACTED] | [LOCAL NETWORK/PORT DETAILS OMITTED] | [HISTORICAL NOTE OMITTED] |

Caddy configuration validation returned `Valid configuration`. That proves syntax/adaptation is valid, not that every upstream is reachable.

The user confirmed that the historical `photos` upstream 502 errors were expected during an intentional Immich image update. They are not recorded as a persistent fault.

## Historical published-host-port inventory omitted## DNS path observation

Pi-hole's active upstream configuration was observed as `8.8.8.8` and `8.8.4.4`. Unbound is running separately on `[DOCKER_NETWORK_5]`, while Pi-hole is on `bridge`. The observed Pi-hole upstream therefore bypasses the running Unbound instance. This may be deliberate, but the intended DNS design has not been confirmed.

## Architecture diagrams

- [Current infrastructure architecture](architecture-overview.md) explains the host, Docker runtime, Caddy routes and persistent storage.
- [Current architecture source](../diagrams/current-architecture.mmd) provides the end-to-end Mermaid diagram.
- [Docker network topology source](../diagrams/docker-network-topology.mmd) maps observed container membership across Docker networks.

The diagrams distinguish configured routes and observed network membership from confirmed application-level communication.
