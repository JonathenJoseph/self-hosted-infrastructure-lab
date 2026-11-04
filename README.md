

# Self-Hosted Infrastructure Lab

A practical infrastructure portfolio documenting the design, operation, troubleshooting and recovery of a self-hosted environment built around Windows 11, Docker Desktop, WSL 2 and a collection of containerised services.

The project focuses on understanding how infrastructure components work together, documenting operational decisions, investigating failures and testing recovery procedures.

It is a **documentation-led portfolio**. It does not contain the live deployment configuration, credentials or private operational evidence.

## Project highlights

### Backup and recovery engineering

The main case study documents an Immich backup and recovery investigation using Restic and PostgreSQL.

Recorded achievements include:

- Implementing encrypted backups to external storage.
- Automating scheduled backups and retention.
- Validating backup inputs and database dump integrity.
- Investigating storage-related backup failures.
- Comparing original and restored database dump hashes using SHA-256.
- Importing a restored PostgreSQL database into an isolated test environment.
- Completing a full Restic data-pack read with 9,221 of 9,221 packs read and no reported errors.

Complete Immich application recovery, including application startup against the restored database and verification of media availability, remains untested.

[Read the backup and recovery case study](docs/case-studies/immich-backup-incident.md).

### Containerisation and infrastructure

The environment uses Docker Desktop on Windows 11, with Ubuntu on WSL 2 acting as a client of the same Docker daemon.

The documented services include:

- Reverse proxying with Caddy.
- Photo and video management with Immich.
- Media services and home automation.
- DNS-related services.
- Self-hosted web applications.
- Remote-access services.
- Supporting infrastructure and monitoring tools.

The inventory is a point-in-time record, not a promise that every service is currently running.

### Networking and remote access

The documentation distinguishes between infrastructure responsibilities:

- **UniFi:** local DNS records and LAN routing.
- **Caddy:** HTTP/HTTPS application-level reverse proxying.
- **Twingate:** remote access configured to reach services across the home network.
- **UniFi Teleport:** a separate remote VPN option through WiFiman.

Twingate and UniFi Teleport are independent remote-access methods. They do not need to be used together.

## Architecture

The environment can be understood in four broad layers:

1. Home networking and remote access.
2. Windows and Docker Desktop host infrastructure.
3. Containerised applications and supporting services.
4. Persistent data and encrypted backup storage.

See the [architecture overview](docs/architecture-overview.md) and [current architecture diagram](diagrams/current-architecture.mmd).

## Repository navigation

| Document | Purpose |
|---|---|
| [Architecture overview](docs/architecture-overview.md) | High-level system design and component responsibilities |
| [Current state](docs/current-state.md) | Recorded environment state and known limitations |
| [Container inventory](docs/container-inventory.md) | Services and their roles |
| [Networking](docs/networking.md) | DNS, routing, proxying and remote access |
| [Security baseline](docs/security-baseline.md) | Security principles, observations and improvement areas |
| [Backup and recovery](docs/backup-and-recovery.md) | Restic design, automation, retention and recovery testing |
| [Backup incident case study](docs/case-studies/immich-backup-incident.md) | Storage investigation and recovery evidence |
| [Evidence and method](docs/evidence-and-method.md) | How observations, tests and limitations are recorded |
| [Known gaps and actions](docs/known-gaps-and-actions.md) | Outstanding technical work and limitations |
| [Security policy](SECURITY.md) | Reporting a security concern |

## Technology overview

| Area | Technology |
|---|---|
| Host operating system | Windows 11 |
| Container platform | Docker Desktop |
| Linux environment | Ubuntu on WSL 2 |
| Reverse proxy | Caddy |
| Photo and video management | Immich |
| Backup software | Restic |
| Database | PostgreSQL |
| Network infrastructure | UniFi |
| Remote access | Twingate and UniFi Teleport |
| Documentation diagrams | Mermaid |

## Evidence and accuracy

This portfolio distinguishes between:

- Directly observed configuration or behaviour.
- Results reported by the system operator.
- Tests that were performed and their recorded outcomes.
- Inferences that have not been independently confirmed.
- Work that has not yet been tested.

A successful database restore is not described as a complete application recovery. A successful backup is not treated as proof that every possible recovery scenario works.

The backup evidence in this repository includes results recorded on 1–2 November 2026. These are historical test records, not a live status feed.

See [Evidence and method](docs/evidence-and-method.md) for the evidence conventions and limitations.

## Scope and publication boundaries

This is a sanitised, documentation-led portfolio.

The repository deliberately excludes:

- Deployment environment files and credentials.
- Private keys and sensitive certificates.
- Raw backup repositories and database dumps.
- Internal addressing and unnecessary network configuration.
- Detailed firewall rules and private access-control configuration.
- Private screenshots, logs and operational records.

Infrastructure names and relationships are presented at a level intended to explain engineering decisions without publishing an unnecessary map of the home network.

## Current limitations

The project is not presented as an enterprise production environment or a fully validated disaster-recovery platform.

Known limitations include:

- Complete Immich application recovery has not been tested.
- The cause of an earlier storage-related backup failure has not been established.
- An earlier Restic check reported additional files whose status remains unresolved.
- Automatic Restic pruning remains disabled.
- Some container security and persistence improvements remain under review.

These limitations are documented rather than hidden.

## Project status

This repository records practical work and its supporting evidence. It should be read as a technical portfolio and an evolving record of an environment, not as a guarantee of current service availability.

