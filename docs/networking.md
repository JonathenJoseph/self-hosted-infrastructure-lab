# Networking and Routing

**Snapshot date:** 1 November 2026

## UniFi local DNS and Caddy web routing

The home network uses the UniFi Gateway to manage local DNS records and hostnames, as well as LAN routing.

A home-network client asks the gateway to resolve a local domain or hostname. The gateway returns the configured address, which in the documented web-service design is the Windows host's LAN address.

Caddy receives HTTP/HTTPS requests and forwards them to configured application services; exact host-published port details are omitted.

The request flow is:

1. The client asks UniFi DNS to resolve the local hostname.
2. UniFi returns the configured host address.
3. The client sends the web request to that address on port 80 or 443.
4. Caddy selects the configured upstream and forwards the request to the application.

**Responsibility split:**

- UniFi provides local name resolution and LAN routing.
- Caddy provides application-level HTTP and HTTPS routing.

UniFi resolves the name; Caddy routes the web request to the application.

## Remote access from outside the home network

Two independent remote-access methods are used.

### Twingate

The Twingate Connector provides remote-access connectivity; historical container network mode is omitted.

The operator has confirmed that Twingate allows remote access to **any service on the home network**. The documented architecture therefore treats Twingate as a general remote-access path, not merely as an entry point to Caddy or a small subset of applications.

The Connector's credentials and private configuration are intentionally excluded from this repository.

### UniFi Teleport VPN

UniFi Teleport is a separate remote-access VPN option using the WiFiman client.

A remote device connects to the UniFi Gateway through Teleport. The gateway then provides VPN access to the home network subject to the configured network policies.

Twingate and Teleport are independent options. They do not need to be active together.

Ubiquiti documentation:

- [UniFi DNS Records and Local Hostnames](https://help.ui.com/hc/en-us/articles/15179064940439-UniFi-DNS-Records-and-Local-Hostnames)
- [UniFi Gateway - Teleport VPN](https://help.ui.com/hc/en-us/articles/5246403561495-UniFi-Gateway-Teleport-VPN)
- [Using WiFiman](https://help.ui.com/hc/en-us/articles/205204150-Using-WiFiman)

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

The historical `photos` upstream 502 errors were explained as expected errors during an intentional Immich image update. They are not recorded as a persistent fault.

## Historical published-host-port inventory omitted## DNS path observation

Pi-hole's active upstream configuration was observed as `8.8.8.8` and `8.8.4.4`. Unbound is running separately on `[DOCKER_NETWORK_5]`, while Pi-hole is on `bridge`. The observed Pi-hole upstream therefore bypasses the running Unbound instance. This may be deliberate, but the intended DNS design has not been confirmed.

## Architecture diagrams

- [Current infrastructure architecture](architecture-overview.md) explains the host, Docker runtime, Caddy routes and persistent storage.
- [Current architecture source](../diagrams/current-architecture.mmd) provides the end-to-end Mermaid diagram.
- [Docker network topology source](../diagrams/docker-network-topology.mmd) maps observed container membership across Docker networks.

The diagrams distinguish configured routes, operator-confirmed remote access and observed network membership from assumptions about application-level communication.
