# Current State

**Snapshot date:** 1 November 2026  
**Platform:** Windows 11 host, Docker Desktop, WSL 2 backend  
**Assessment mode:** read-only discovery

## Host baseline

| Component | Observed state | Notes |
|---|---|---|
| Physical host | HP Elite Mini 800 G9 | Model reported during host discovery |
| CPU | Intel Core i9-13900 | 24 cores / 32 logical processors reported |
| Memory | Approximately 31.7 GiB | Based on reported physical-memory bytes |
| System volume | Approximately 1 TB NTFS | Free-space readings varied between discovery runs; refresh before capacity planning |
| Windows build | 26200.9457 | Build family corresponds to Windows 11; registry product-name field was inconsistent and should not be used alone |
| Docker Desktop | 4.95.0 | Installed on Windows host |
| Docker Engine | 29.8.2 | Linux/amd64 server reported by Docker |
| Docker context | `desktop-linux` | Active Windows CLI context |
| WSL | 3.0.3.0 | WSL 2 backend in use |
| WSL kernel | 6.18.54.1-1 | Reported by `wsl --version` |
| Default WSL distribution | Ubuntu | WSL 2 |

The machine has ample CPU capability for the observed workload. Capacity planning still needs current disk and memory measurements under representative load; a single `docker stats` sample is not a benchmark.

## Docker daemon topology

The Windows Docker CLI and the Docker CLI inside Ubuntu WSL were both observed to communicate with the same Docker Desktop daemon. The Docker server ID matched in both environments. Ubuntu WSL is therefore a client/workspace for the same engine, not a second independent Docker host.


## Container estate summary

The current inventory snapshot records 16 running containers within the scope of this portfolio. See [Container Inventory](container-inventory.md) for the documented services and their observed configuration.

The in-scope workloads include:

- Reverse proxy: Caddy
- Photos and videos: Immich and its database, machine-learning and cache services
- Home automation: Home Assistant
- Media: Jellyfin
- Search: SearXNG
- Dashboard: Homepage
- File conversion: Vert
- AI web interface: Open WebUI
- Apprenticeship Tracker: web and API containers
- DNS: Pi-hole and Unbound
- Remote access connector: Twingate

See [Container Inventory](container-inventory.md) for per-container detail.

## Deployment management

Six relevant Compose projects were reported by `docker compose ls`, including projects that contain only part of a related service set. Five Windows-hosted Compose definitions were successfully validated during discovery. One expected Pi-hole Compose definition was not present at the path recorded in the surviving Unbound metadata.

Six in-scope containers were not associated with Compose project labels in the observed metadata:

- Home Assistant
- SearXNG
- Jellyfin
- Open WebUI
- Pi-hole
- Twingate Connector

This does not prove these containers were created manually, but their current Compose ownership or reproducible deployment definition has not been established.

The Pi-hole/Unbound deployment has an additional inconsistency: Unbound metadata references a Compose file under the Pi-hole directory, but that Compose file was not present when checked. The active Pi-hole container itself had no Compose labels.

## Operating principles for this documentation

1. Treat command output as evidence, not as permission to change configuration.
2. Preserve application data before recreating a container.
3. Do not publish secrets or certificate private keys.
4. Do not prune unused images or volumes until ownership and recovery requirements are understood.
5. Keep point-in-time observations dated and refresh them before operational decisions.
