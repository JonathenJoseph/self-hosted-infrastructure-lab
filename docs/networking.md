# Networking and Routing

**Last updated:** 2 November 2026

## 1. Network architecture

The home network uses UniFi networking equipment for local DNS, hostname resolution and LAN routing.

Docker Desktop runs the hosted applications on the Windows 11 host.

Caddy provides application-level HTTP and HTTPS routing.

Remote access is provided through two independent options:

- Twingate.
- UniFi Teleport VPN through WiFiman.

## 2. UniFi local DNS and hostname resolution

The UniFi Gateway manages local DNS records and hostname resolution.

A home-network client asks the gateway to resolve a configured local hostname. The gateway returns the configured address, which points to the relevant host LAN address in this web-service architecture.

This allows local clients to use meaningful hostnames rather than memorising IP addresses and ports.

The gateway also handles LAN routing.

The actual internal hostnames, IP addresses and local domain are deliberately omitted from this repository.

## 3. Caddy reverse proxy

Caddy receives HTTP and HTTPS requests on the Windows host's published TCP ports 80 and 443.

It then selects a configured upstream based on the requested site.

The functional request path is:

1. Client requests a local hostname.
2. UniFi resolves the hostname.
3. Client connects to the host address.
4. Caddy receives the HTTP/HTTPS request.
5. Caddy forwards the request to the application.

UniFi DNS and Caddy have separate responsibilities. UniFi resolves names; Caddy selects the application endpoint.

## 4. Remote access

### Twingate

The Twingate Connector provides remote-access connectivity; historical container network mode is omitted.

The operator has confirmed that Twingate allows access to any service on the home network.

This is recorded as user-confirmed configuration information.

It is not restricted in this documentation to Caddy-hosted applications or a small list of web services.

Twingate's credentials, tokens and private configuration are excluded from this repository.

### UniFi Teleport VPN

UniFi Teleport is a separate remote-access VPN option through the WiFiman client.

A remote device connects to the UniFi Gateway through Teleport. Access then follows the gateway's configured network policies.

Teleport and Twingate are independent remote-access options. They do not need to be active together.

Official references:

- [UniFi DNS Records and Local Hostnames](https://help.ui.com/hc/en-us/articles/15179064940439-UniFi-DNS-Records-and-Local-Hostnames)
- [UniFi Gateway - Teleport VPN](https://help.ui.com/hc/en-us/articles/5246403561495-UniFi-Gateway-Teleport-VPN)
- [Using WiFiman](https://help.ui.com/hc/en-us/articles/205204150-Using-WiFiman)

## 5. Docker networks

The captured Docker network inventory includes nine networks:

- `[DOCKER_NETWORK_3]`
- `bridge`
- `[DOCKER_NETWORK_1]`
- `[DOCKER_NETWORK_4]`
- `host`
- `[DOCKER_NETWORK_2]`
- `none`
- `[DOCKER_NETWORK_5]`
- `vems_default`

Observed memberships:

| Network | Containers observed |
|---|---|
| `[DOCKER_NETWORK_1]` | Caddy, Apprenticeship Tracker web, Home Assistant and Jellyfin |
| `[DOCKER_NETWORK_2]` | Caddy, Immich server, PostgreSQL, machine learning, Valkey and Homepage |
| `[DOCKER_NETWORK_3]` | Apprenticeship Tracker web and API |
| `[DOCKER_NETWORK_4]` | Homepage |
| `bridge` | Home Assistant, SearXNG, Jellyfin, Open WebUI and Pi-hole |
| `[DOCKER_NETWORK_5]` | Unbound |
| `vems_default` | VEMS |
| `host` | Twingate Connector |

The `host` and `none` entries are Docker network modes and special networks rather than ordinary application networks.

A container may belong to multiple Docker networks. Membership indicates potential network connectivity, not confirmed application traffic.

## 6. Caddy route inventory

Nine site snippets were observed.

| Site snippet | Configured upstream | Observation |
|---|---|---|
| [REVERSE-PROXY ROUTE] | [UPSTREAM DETAILS REDACTED] | [LOCAL NETWORK/PORT DETAILS OMITTED] | [HISTORICAL NOTE OMITTED] |
| [REVERSE-PROXY ROUTE] | [UPSTREAM DETAILS REDACTED] | [LOCAL NETWORK/PORT DETAILS OMITTED] | [HISTORICAL NOTE OMITTED] |
| [REVERSE-PROXY ROUTE] | [UPSTREAM DETAILS REDACTED] | [LOCAL NETWORK/PORT DETAILS OMITTED] | [HISTORICAL NOTE OMITTED] |
| [REVERSE-PROXY ROUTE] | [UPSTREAM DETAILS REDACTED] | [LOCAL NETWORK/PORT DETAILS OMITTED] | [HISTORICAL NOTE OMITTED] |
| [REVERSE-PROXY ROUTE] | [UPSTREAM DETAILS REDACTED] | [LOCAL NETWORK/PORT DETAILS OMITTED] | [HISTORICAL NOTE OMITTED] |
| [REVERSE-PROXY ROUTE] | [UPSTREAM DETAILS REDACTED] | [LOCAL NETWORK/PORT DETAILS OMITTED] | [HISTORICAL NOTE OMITTED] |
| [REVERSE-PROXY ROUTE] | [UPSTREAM DETAILS REDACTED] | [LOCAL NETWORK/PORT DETAILS OMITTED] | [HISTORICAL NOTE OMITTED] |
| [REVERSE-PROXY ROUTE] | [UPSTREAM DETAILS REDACTED] | [LOCAL NETWORK/PORT DETAILS OMITTED] | [HISTORICAL NOTE OMITTED] |
| [REVERSE-PROXY ROUTE] | [UPSTREAM DETAILS REDACTED] | [LOCAL NETWORK/PORT DETAILS OMITTED] | [HISTORICAL NOTE OMITTED] |

Caddy configuration validation passed.

The Kali route remained unresolved because no matching Windows listener or published container port was identified during the earlier audit.

The Ollama route label does not establish that the upstream is the Ollama API. The captured port belongs to Open WebUI.

## 7. Published host ports

The captured Docker inventory showed the following host bindings:

| Service | Host port | Container port | Protocol |
|---|---:|---:|---|
| Caddy | 80, 443 | 80, 443 | TCP |
| Immich | 2283 | 2283 | TCP |
| Apprenticeship Tracker API | 4000 | 4000 | TCP |
| Home Assistant | 8123 | 8123 | TCP |
| SearXNG | 8080 | 8080 | TCP |
| Jellyfin | 8096 | 8096 | TCP |
| Homepage | 3001 | 3000 | TCP |
| Vert | 3005 | 80 | TCP |
| Open WebUI | 3000 | 8080 | TCP |
| Pi-hole DNS | 15353 | 53 | TCP and UDP |
| Pi-hole web interface | 8081 | 80 | TCP |
| Unbound | 5335 | 53 | UDP |

A Docker binding on all host interfaces does not by itself prove internet exposure. Windows Firewall, router policy, network segmentation, VPN configuration and NAT also influence reachability.

## 8. DNS architecture finding

Pi-hole was observed using these configured upstream DNS servers:

- `8.8.8.8`
- `8.8.4.4`

Unbound was running separately on `[DOCKER_NETWORK_5]`.

Pi-hole was attached to the default `bridge` network rather than `[DOCKER_NETWORK_5]`, and its configured upstreams bypassed the Unbound container.

This may be intentional, but the intended DNS design has not been confirmed.

## 9. Networking risks and actions

| Finding | Implication | Action |
|---|---|---|
| Multiple host-published ports | Broad listening scope | Confirm intended exposure of each service |
| Pi-hole and Unbound separation | Unbound not used by Pi-hole | Confirm intended DNS architecture |
| Unresolved Kali upstream | Potential broken route | Identify intended target or remove route |
| Open WebUI labelled as Ollama | Potentially misleading route | Confirm whether the route name is intentional |
| Pi-hole has no persistent mounts | State may be lost on container recreation | Export configuration and plan persistent storage |

## Related diagrams and documents

- [Current Architecture](architecture-overview.md)
- [Current Architecture Mermaid Source](../diagrams/current-architecture.mmd)
- [Docker Network Topology](../diagrams/docker-network-topology.mmd)
- [Container Inventory](container-inventory.md)
- [Security Baseline](security-baseline.md)
