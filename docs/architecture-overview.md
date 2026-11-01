# Architecture Overview

**Snapshot date:** 1 November 2026  
**Scope:** Current Windows 11 and Docker Desktop environment; one separately deferred project is excluded.

## Purpose

This page explains how the documented services are hosted, how requests reach them, how Docker networking is arranged, and where important application data is stored. It describes the observed environment, not a target design.

## Architecture diagram

GitHub renders the Mermaid diagram below. The editable source is also available at [`diagrams/current-architecture.mmd`](../diagrams/current-architecture.mmd).

```mermaid
flowchart TB
    subgraph REMOTE["Remote access outside the home"]
        REMOTE_DEVICE["Remote device"]
        TW_CLIENT["Twingate client"]
        WIFI_CLIENT["WiFiman client - Teleport VPN"]
    end

    subgraph HOME_NET["Home network - UniFi"]
        UGW["UniFi Gateway - local DNS and LAN routing"]
        LAN_CLIENT["Home network clients"]
        SWITCH["UniFi switch and access points"]
        TW_RESOURCES["Twingate authorised resources - exact list not documented"]
    end

    subgraph HOST["HP Elite Mini 800 G9 - Windows 11 host"]
        FW["Windows Defender Firewall"]
        subgraph WSL["WSL 2"]
            UBUNTU["Ubuntu - Docker CLI client"]
        end
        subgraph DOCKER["Docker Desktop - Linux Docker Engine"]
    CADDY["Caddy reverse proxy"]
    TWIN["Twingate Connector"]
            subgraph APPGROUPS["Application containers"]
                WEB["Apprenticeship Tracker web"]
                HA["Home Assistant"]
                JELLY["Jellyfin"]
                HOMEPAGE["Homepage"]
                IMMICH["Immich server"]
                PG["Immich PostgreSQL"]
                ML["Immich machine learning"]
                VALKEY["Valkey"]
                SEARCH["SearXNG"]
                VERT["Vert"]
                WEBUI["Open WebUI"]
                API["Apprenticeship Tracker API - host port 4000"]
                PI["Pi-hole DNS service"]
                UNBOUND["Unbound - host UDP port 5335"]
                KALI["Kali route target - not identified"]
            end
        end
    end

    %% UniFi resolves local names; Caddy handles HTTP and HTTPS application routing.
    LAN_CLIENT -->|DNS query for local hostname| UGW
    UGW -.->|Local DNS answer points to host LAN address| LAN_CLIENT
    LAN_CLIENT --> SWITCH
    SWITCH -->|HTTP or HTTPS request to host address| FW
    FW --> CADDY

    %% Remote access paths are independent. Resource-level Twingate rules were not captured.
    REMOTE_DEVICE --> TW_CLIENT
    TW_CLIENT -->|Private access connection| TWIN
    TWIN -.->|Resource access depends on Twingate policy| TW_RESOURCES

    REMOTE_DEVICE --> WIFI_CLIENT
    WIFI_CLIENT -->|Teleport VPN connection| UGW
    UGW -->|Permitted access to home network| FW

    UBUNTU -.->|Uses the same Docker daemon| DOCKER

    %% Caddy upstreams observed in the current configuration.
    %% Historical Caddy upstream and port mapping omitted
    %% Historical Caddy upstream and port mapping omitted
    %% Historical Caddy upstream and port mapping omitted
    %% Historical Caddy upstream and port mapping omitted
    %% Historical Caddy upstream and port mapping omitted
    %% Historical Caddy upstream and port mapping omitted
    %% Historical Caddy upstream and port mapping omitted
    %% Historical Caddy upstream and port mapping omitted
    %% Historical Caddy upstream and port mapping omitted
    IMMICH --> PG
    IMMICH --> ML
    IMMICH --> VALKEY

    subgraph STORAGE["Persistent storage"]
    HOSTDATA["Host storage (mount details omitted)"]
        VOLUMES["Docker named volumes - model cache and application state"]
        EXTERNAL["External Seagate USB drive - Restic repository has zero completed snapshots"]
    end

    HOSTDATA -.->|Bind mounts| IMMICH
    %% Historical host-mount relationship omitted
    VOLUMES -.->|Model cache| ML
    VOLUMES -.->|Application data| WEBUI
    VOLUMES -.->|Caddy configuration and state| CADDY
    %% Historical host-mount relationship omitted
    HOSTDATA -.->|Configuration| HA
    %% Historical host-mount relationship omitted
    IMMICH -.->|Backup attempt failed - no snapshot| EXTERNAL

    PI -.->|Configured upstreams| PUBLIC_DNS["Public DNS upstreams"]
    PI --- UNBOUND_NOTE["Unbound is running separately and is not configured as Pi-hole upstream"]

    classDef unresolved stroke-dasharray: 5 5;
    class KALI,EXTERNAL,TW_RESOURCES unresolved;
```

## 1. Host and container runtime

The current environment has three relevant layers:

| Layer | Observed component | Role |
|---|---|---|
| Physical host | HP Elite Mini 800 G9 | Runs Windows and the local infrastructure workload |
| Host operating system | Windows 11 | Owns the physical network interfaces, host firewall and NTFS-backed application directories |
| Container platform | Docker Desktop 4.95.0 | Provides the Linux container environment and Docker Engine 29.8.2 |
| Linux workspace | Ubuntu on WSL 2 | Provides a Linux command-line workspace and Docker CLI; it connects to the same Docker Desktop daemon as the Windows CLI |

Ubuntu WSL is not a second independent Docker host in this setup. The Docker server ID reported by the Windows and Ubuntu clients matched during discovery.

## 2. Network naming, local access and remote access

The documented network design has separate responsibilities at the DNS, gateway and application layers:

- **UniFi Gateway:** handles local DNS records and hostname resolution for the home network, as well as LAN routing. A UniFi local hostname resolves for clients that use the UniFi Gateway as their DNS server. The gateway's DNS function resolves the name; it does not perform Caddy's application-level web routing.
- **Caddy:** receives HTTP/HTTPS requests on the Windows host's published ports 80 and 443 and reverse-proxies them to the configured containers or host-published service ports. In this design, UniFi resolves the local domain or hostname to the host's LAN address, then Caddy routes the web request to the relevant application.
- **Remote access:** there are two independent ways to connect from outside the home network: Twingate, using its client and the locally running Twingate Connector; or UniFi Teleport VPN, using the WiFiman client. They are separate options and do not need to be active together.

The Twingate Connector was observed using Docker host networking. The exact Twingate resource definitions and independent off-site reachability tests were not included in the captured evidence, so the diagram does not claim that every service is exposed through Twingate. UniFi documents Teleport as its remote-access VPN and supports it through WiFiman Mobile and WiFiman Desktop. Whether each remote-access path resolves the same local hostnames and can reach each application should be verified from an off-site client before being described as tested.

## 3. Caddy request and service flow

Caddy provides the HTTP/HTTPS reverse-proxy entry point; exact host-published port details are omitted.

Two configured routes use Docker service-name resolution on `[DOCKER_NETWORK_1]`:

- Apprenticeship Tracker web: `web:80`
- Home Assistant: `homeassistant:8123`

The other identified Caddy routes use `[DOCKER_HOST_GATEWAY_ALIAS]` and a host-published port:

| Application route label | Configured upstream | Identified destination |
|---|---|---|
| Dashboard | `[DOCKER_HOST_GATEWAY_ALIAS]:3001` | Homepage |
| Movies | `[DOCKER_HOST_GATEWAY_ALIAS]:8096` | Jellyfin |
| Photos | `[DOCKER_HOST_GATEWAY_ALIAS]:2283` | Immich server |
| Search | `[DOCKER_HOST_GATEWAY_ALIAS]:8080` | SearXNG |
| Vert | `[DOCKER_HOST_GATEWAY_ALIAS]:3005` | Vert |
| Ollama | `[DOCKER_HOST_GATEWAY_ALIAS]:3000` | Open WebUI host port in the captured inventory; this route name does not establish that it serves the Ollama API |
| Kali | `[DOCKER_HOST_GATEWAY_ALIAS]:8088` | Unresolved; no matching Windows listener or published container port was found during the check |

Caddy configuration validation passed. That confirms the configuration can be parsed and adapted; it does not prove that every upstream is reachable at all times.

The user confirmed that the logged Immich 502 responses occurred during an intentional Immich image update. They are recorded as maintenance-related downtime, not as a persistent route fault.

## 4. Application groups

### Immich

The observed Immich application group contains:

- `[CONTAINER_ROLE_IMMICH]`, the application server;
- `[CONTAINER_ROLE_DATABASE]`, the PostgreSQL database;
- `[CONTAINER_ROLE_PROCESSOR]`, the machine-learning service;
- `[CONTAINER_ROLE_CACHE]`, which uses the Valkey image in the captured inventory.

These containers share `[DOCKER_NETWORK_2]`. Caddy and Homepage also join that network. The configured Caddy photos route currently uses the host-published Immich port rather than the direct Docker network address.

### Apprenticeship Tracker

The observed project has separate web and API containers. The web container shares `[DOCKER_NETWORK_1]` with Caddy and is configured as the direct `web:80` upstream. The API publishes host port 4000. The API and web containers are also members of the project's default network.

### Other services

The repository documents several application services; historical per-service routes and published-port mappings are omitted.

The historical DNS upstream and Docker-network relationship is omitted.

The Twingate Connector provides remote-access connectivity; historical container network mode is omitted.

## Historical Docker network topology details omitted## 6. Persistent data and storage

Application state uses persistent storage; historic per-service host mount details are omitted.

| Workload | Observed storage pattern | Documentation concern |
|---|---|---|
| [SERVICE ROLE] | [HISTORICAL STORAGE TOPOLOGY OMITTED] | [OPERATIONAL DETAILS GENERALISED] |
| [SERVICE ROLE] | [HISTORICAL STORAGE TOPOLOGY OMITTED] | [OPERATIONAL DETAILS GENERALISED] |
| [SERVICE ROLE] | [HISTORICAL STORAGE TOPOLOGY OMITTED] | [OPERATIONAL DETAILS GENERALISED] |
| Immich machine learning | Docker named volume | Model cache rather than primary user media |
| [SERVICE ROLE] | [HISTORICAL STORAGE TOPOLOGY OMITTED] | [OPERATIONAL DETAILS GENERALISED] |
| [SERVICE ROLE] | [HISTORICAL STORAGE TOPOLOGY OMITTED] | [OPERATIONAL DETAILS GENERALISED] |
| Open WebUI | Docker named volume at `/app/backend/data` | Preserve application data volume |
| [SERVICE ROLE] | [HISTORICAL STORAGE TOPOLOGY OMITTED] | [OPERATIONAL DETAILS GENERALISED] |

The external Restic repository was initialised on a Seagate USB drive. The first full backup attempt failed during writes with Windows disk and NTFS errors. Restic later reported zero completed snapshots and its repository integrity check found no structural errors. The repository is **not a completed backup** and must not be represented as a verified recovery copy.

## Historical host-port and firewall-boundary details omitted## 8. Diagram legend and limitations

- **Solid arrows** represent a configured route or an observed service relationship documented from configuration/inventory.
- **Dashed arrows** represent unresolved or unverified relationships.
- The network topology diagram records container membership, not application traffic.
- The diagram intentionally omits internal IP addresses, private hostnames, credentials and certificate material.
- The architecture is a point-in-time view. It should be refreshed when containers, Compose definitions, ports, networks or storage mounts change.

## Related documentation

- [Container Inventory](container-inventory.md)
- [Networking and Routing](networking.md)
- [Backup and Recovery](backup-and-recovery.md)
- [Security Baseline](security-baseline.md)
- [Evidence and Method](evidence-and-method.md)
