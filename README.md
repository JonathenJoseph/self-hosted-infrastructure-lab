# Self-Hosted Infrastructure Lab

> A student-built infrastructure portfolio exploring containerisation, networking, systems administration, security and data protection.

This repository documents a practical home lab that I build, operate, troubleshoot and improve alongside my studies.

It demonstrates hands-on experience with Docker, Windows, Linux, networking, reverse proxies, remote access, persistent storage, backup automation and recovery testing.

The portfolio is designed to demonstrate skills relevant to IT infrastructure, systems administration, networking, DevOps and junior platform operations roles.

This is a personal learning environment, not an enterprise production deployment. Technical claims are supported by observed configuration, test results or explicitly identified operator confirmation.

---

## Infrastructure overview

| Component | Technology |
|---|---|
| Physical host | HP Elite Mini 800 G9 |
| Operating system | Windows 11 |
| Container platform | Docker Desktop |
| Linux environment | Ubuntu on WSL 2 |
| Container engine | Docker Engine |
| Reverse proxy | Caddy |
| Local DNS and LAN routing | UniFi Gateway |
| Remote access | Twingate and UniFi Teleport through WiFiman |
| Backup software | Restic |
| Database | PostgreSQL 14 |
| Backup destination | External Seagate storage |
| Automation | PowerShell and Windows Task Scheduler |
| Documentation | Markdown and Mermaid |

## Network and remote access

The network architecture separates local name resolution, application routing and remote connectivity.

- **UniFi Gateway:** handles local DNS records, hostname resolution and LAN routing.
- **Caddy:** receives HTTP/HTTPS requests and reverse-proxies them to configured applications.
- **Twingate:** provides remote access to any service on the home network, as confirmed by the operator.
- **UniFi Teleport:** provides an independent remote VPN access option through WiFiman.

Twingate and Teleport are separate remote-access methods and do not need to operate together.

See [Architecture Overview](docs/architecture-overview.md) and [Networking and Routing](docs/networking.md).

## Hosted services

The documented environment includes:

| Category | Services |
|---|---|
| Reverse proxy | Caddy |
| Photos and videos | Immich |
| Database | PostgreSQL |
| Home automation | Home Assistant |
| Media management | Jellyfin |
| Search | SearXNG |
| Dashboard | Homepage |
| File conversion | Vert |
| AI interface | Open WebUI |
| DNS | Pi-hole and Unbound |
| Remote access | Twingate Connector |
| Application development | Apprenticeship Tracker |
| Other projects | VEMS and supporting services |

The container inventory documents service images, ports, health checks, networks and storage observations.

## Selected technical achievements

### Docker and infrastructure discovery

- Inspected containers, images, persistent volumes, mounts and network memberships.
- Mapped Compose projects and identified services without recoverable deployment definitions.
- Distinguished the Ubuntu WSL Docker CLI from the Docker Desktop daemon it controls.
- Investigated container health, restart behaviour and service dependencies.

### Network architecture

- Documented UniFi local DNS and LAN routing.
- Mapped Caddy reverse-proxy routes to Docker services and host-published ports.
- Reviewed Docker bridge network membership and exposed host ports.
- Documented two independent remote-access paths through Twingate and UniFi Teleport.

### Backup and recovery engineering

Established an encrypted Restic repository on external storage and completed an initial full backup.

The initial snapshot:

- Processed 48,335 files.
- Scanned 197.721 GiB.
- Stored 135.725 GiB in the repository.
- Completed with exit code `0`.

A subsequent daily snapshot was also created.

### Recovery testing

The recovery process was tested beyond simply checking that a backup job completed.

- Restored a PostgreSQL database dump from Restic.
- Compared the original and restored file using SHA-256.
- Confirmed that both files were byte-for-byte identical.
- Imported the recovered dump into an isolated PostgreSQL 14 container.
- Verified 72 base tables and eight PostgreSQL extensions.
- Removed the temporary test container and volume without modifying the live Immich database.

This establishes a tested database restoration path. A complete Immich application recovery remains outstanding.

### Backup automation

Created a PowerShell backup script and configured Windows Task Scheduler to run it daily at 05:00.

The script includes:

- External drive checks.
- Existing repository validation.
- Database dump freshness and gzip integrity checks.
- Windows Volume Shadow Copy integration.
- Snapshot verification.
- Logging and failure handling.
- Automatic snapshot retention after successful backup verification.

The first scheduled execution using the latest retention-enabled script is still awaiting confirmation.

## Selected case studies

| Case study | Technical focus |
|---|---|
| [Immich backup failure and recovery](docs/case-studies/immich-backup-incident.md) | Storage troubleshooting, Restic, PostgreSQL recovery and automation |
| [Architecture overview](docs/architecture-overview.md) | Host architecture, application services and data flow |
| [Security baseline](docs/security-baseline.md) | Firewall configuration, container privileges and persistent-state risks |
| [Networking and routing](docs/networking.md) | UniFi DNS, Docker networks, Caddy and remote access |

## Repository documentation

| Document | Purpose |
|---|---|
| [Current State](docs/current-state.md) | Point-in-time infrastructure and operational status |
| [Architecture Overview](docs/architecture-overview.md) | Host, container and storage architecture |
| [Container Inventory](docs/container-inventory.md) | Container images, ports, health and persistence |
| [Networking and Routing](docs/networking.md) | UniFi, Twingate, Teleport, Docker networking and Caddy |
| [Backup and Recovery](docs/backup-and-recovery.md) | Backup implementation, verification and recovery testing |
| [Security Baseline](docs/security-baseline.md) | Security findings and recommendations |
| [Known Gaps and Actions](docs/known-gaps-and-actions.md) | Outstanding work and technical risks |
| [Evidence and Method](docs/evidence-and-method.md) | Discovery methods and confidence classifications |
| [Immich Backup Case Study](docs/case-studies/immich-backup-incident.md) | Detailed technical investigation |
| [Current Architecture Diagram](diagrams/current-architecture.mmd) | Mermaid architecture source |
| [Docker Network Topology](diagrams/docker-network-topology.mmd) | Docker network membership |

## Working approach

The portfolio follows an evidence-led approach:

- **Verified:** supported by command output or an inspected configuration.
- **User-confirmed:** explicitly confirmed by the operator.
- **Inferred:** supported by observations but not directly demonstrated.
- **Open:** requires further testing or confirmation.
- **Deferred:** intentionally outside the current scope.

This distinction prevents configuration observations from being presented as proven operational outcomes.

## Security and privacy

This is a sanitised portfolio.

It intentionally excludes:

- Passwords and API keys.
- Private keys and certificate contents.
- Raw database dumps.
- Restic repository files and passwords.
- Unredacted environment files.
- Private network addresses and sensitive hostnames.

The repository is being maintained privately during review. Any public release will receive a separate security and privacy review.
