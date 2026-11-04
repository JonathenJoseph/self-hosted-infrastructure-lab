

# Container Inventory

## 1. Purpose

This document records the main services observed in the self-hosted environment and explains their roles.

It is a documentation snapshot, not a live inventory or availability report.

Container names, configuration, network membership and service state may change. The public version intentionally omits detailed host-port mappings and internal Docker network names.

## 2. Platform

The environment uses Docker Desktop on a Windows 11 host.

Ubuntu on WSL 2 is used as an additional Docker CLI environment and communicates with the same Docker Desktop daemon.

The container inventory should therefore be understood as one Docker environment accessed from more than one command-line context, not multiple independent Docker engines.

## 3. Application and infrastructure services

| Service | Role | Category |
|---|---|---|
| Caddy | HTTP/HTTPS reverse proxy | Networking |
| Immich Server | Photo and video management application | Application |
| Immich PostgreSQL | Application database | Data |
| Immich Machine Learning | Supporting media-processing service | Application support |
| Immich Valkey | Supporting application service | Data |
| Apprenticeship Tracker Web | Web interface for a self-hosted project | Application |
| Apprenticeship Tracker API | Backend application interface | Application |
| Home Assistant | Home automation | Home infrastructure |
| SearXNG | Self-hosted search metasearch service | Utility |
| Jellyfin | Media streaming | Media |
| Homepage | Service dashboard | Management |
| Vert | File conversion utility | Utility |
| Open WebUI | Web interface for AI services | Application |
| Pi-hole | DNS filtering | Networking |
| Unbound | Recursive DNS service | Networking |
| Twingate Connector | Remote-access connectivity | Remote access |
| VEMS and supporting services | Supporting application environment | Application |

This list describes the services included in the reviewed portfolio inventory. It does not claim that every service is currently running.

## 4. Service relationships

The architecture includes several logical groups:

- Caddy provides application-level routing to configured upstream services.
- Immich uses a supporting database and application services.
- The Apprenticeship Tracker separates its web interface and API.
- Pi-hole and Unbound are separate DNS-related services.
- Twingate provides remote access independently of the Caddy reverse-proxy design.

The exact Docker network names and individual membership mappings have been omitted from the public inventory.

## 5. DNS distinction

Pi-hole and Unbound were observed as separate services.

At the time of the reviewed configuration, Pi-hole used public DNS resolver services as its upstream resolvers. Unbound was running independently and was not configured as the Pi-hole upstream in the observed arrangement.

The presence of Unbound should therefore not be interpreted as evidence that Pi-hole was forwarding queries through it.

## 6. Operational and security observations

The reviewed environment raised several areas for continued attention:

### Docker API access

A dashboard container used read-only Docker socket access.

Read-only access does not automatically make Docker API access low risk. The Docker API can expose substantial information and capabilities, so socket access should be treated as a privileged integration.

### Persistent configuration

The reviewed Pi-hole container did not have dedicated persistent mounts for its application state. Its state was therefore dependent on the container's writable layer at that time.

A recreation or replacement could affect configuration persistence unless the state is backed up or moved to persistent storage.

### Published services

Some containers published host ports on all interfaces during the review.

This does not prove that those services were accessible from the public internet. Exposure depends on the host firewall, network design, gateway configuration and any upstream controls.

### Application routing

Caddy configuration contained routes to several upstreams. Configuration presence does not establish that every upstream was reachable or healthy during the review.

## 7. Inventory limitations

This document deliberately does not provide:

- Exact host-port mappings.
- Internal IP addresses.
- Detailed Docker network membership.
- Private environment variables.
- Connector configuration.
- Firewall rules.
- Container credentials.
- Private application URLs.

For the high-level relationships, see [architecture overview](architecture-overview.md) and [networking](networking.md).

