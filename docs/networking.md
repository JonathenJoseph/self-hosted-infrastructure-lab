# Networking and Routing

**Snapshot date:** 1 November 2026

## UniFi local DNS and Caddy web routing

The home network uses the UniFi Gateway to manage local DNS records and hostnames. A home-network client asks the gateway to resolve a local domain or hostname. The gateway returns the configured address, which in the documented web-service design is the Windows host's LAN address. UniFi local hostnames resolve only for clients using the UniFi Gateway as their DNS server.

Caddy receives HTTP/HTTPS requests and forwards them to configured application services; exact host-published port details are omitted.

1. The client asks UniFi DNS to resolve the local hostname.
2. UniFi returns the configured host address.
3. The client sends the web request to that address on port 80 or 443.
4. Caddy selects the configured upstream and forwards the request to the application.

This distinction matters: UniFi provides local name resolution and LAN routing; Caddy provides application-level HTTP and HTTPS routing.

## Remote access from outside the home network

Two independent remote-access methods are available in the documented design:

- **Twingate:** a remote device uses the Twingate client to connect through the Twingate Connector running with Docker host networking. The exact private-resource definitions and off-site tests were not captured in this audit. Do not assume that every container or local domain is available through Twingate.
- **UniFi Teleport VPN:** a remote device uses the WiFiman client to connect to the UniFi Gateway. Ubiquiti documents Teleport as a remote-access VPN, with support in WiFiman Mobile and WiFiman Desktop. The gateway then provides access subject to the network's configured policies.

These are separate access options; the user does not need to run both for the same connection. Ubiquiti documents that Teleport updates the client's DNS settings to use the connected UniFi Gateway. This does not, by itself, prove that every local hostname or application is reachable in this particular environment. Off-site DNS and application reachability should be tested before being marked as verified.

References:

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

The user confirmed that the historical `photos` upstream 502 errors were expected during an intentional Immich image update. They are not recorded as a persistent fault.

## Historical published-host-port inventory omitted## DNS path observation

Pi-hole's active upstream configuration was observed as `8.8.8.8` and `8.8.4.4`. Unbound is running separately on `[DOCKER_NETWORK_5]`, while Pi-hole is on `bridge`. The observed Pi-hole upstream therefore bypasses the running Unbound instance. This may be deliberate, but the intended DNS design has not been confirmed.

## Architecture diagrams

- [Current infrastructure architecture](architecture-overview.md) explains the host, Docker runtime, Caddy routes and persistent storage.
- [Current architecture source](../diagrams/current-architecture.mmd) provides the end-to-end Mermaid diagram.
- [Docker network topology source](../diagrams/docker-network-topology.mmd) maps observed container membership across Docker networks.

The diagrams distinguish configured routes and observed network membership from confirmed application-level communication.
