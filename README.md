# Self-Hosted Infrastructure Lab

> **A student-built infrastructure portfolio exploring containerisation, networking, service operations, security and data protection.**

This repository documents a practical home lab that I build, operate and investigate alongside my studies. It records hands-on learning: deploying services, understanding how they communicate, reviewing how data is stored, diagnosing problems and turning the results into clear technical documentation.

The portfolio is intended to demonstrate practical skills relevant to **IT infrastructure, systems administration, networking, DevOps and cybersecurity**. It is not presented as an enterprise production environment. Findings are evidence-based, limitations are recorded, and unfinished work is labelled honestly.

---

## At a glance

| Area | Current environment |
|---|---|
| Host platform | Windows 11 |
| Container platform | Docker Desktop with WSL 2 backend |
| Linux environment | Ubuntu on WSL 2 |
| Container engine | Docker Engine, Linux containers |
| Reverse proxy | Caddy |
| Local DNS and LAN routing | UniFi Gateway |
| Remote access | Twingate and UniFi Teleport through WiFiman |
| Application deployment | Docker Compose and standalone containers |
| Data protection | Encrypted Restic repository with a successful initial snapshot |
| Documentation | Markdown, command-line evidence and Mermaid diagrams |

The current inventory documents **16 running containers** across application, infrastructure and supporting services. Versions and operational details are point-in-time observations and should be refreshed as the lab changes.

## Architecture at a glance

```mermaid
flowchart TB
    HOME_CLIENT["Home network clients"] --> DNS["UniFi Gateway - local DNS and LAN routing"]
    DNS -->|Local hostname resolves to host address| CADDY["Caddy reverse proxy - HTTP and HTTPS"]
    CADDY --> APPS["Self-hosted applications"]

    REMOTE["Remote device"] --> TWIN_CLIENT["Twingate client"]
    TWIN_CLIENT --> TWIN["Twingate Connector"]
    TWIN -->|Access to any home-network service| APPS

    REMOTE --> WIFI["WiFiman - UniFi Teleport VPN"]
    WIFI --> DNS

    HOST["Windows 11 host"] --> DOCKER["Docker Desktop - Linux Docker Engine"]
    WSL["Ubuntu on WSL 2 - Docker CLI"] -. "same Docker daemon" .-> DOCKER
    DOCKER --> CADDY

    IMMICH["Immich"] --> RESTIC["External Restic repository"]
    RESTIC --> SNAPSHOT["Initial snapshot 7ec7a616 saved"]
