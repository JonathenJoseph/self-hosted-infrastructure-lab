

# Architecture Overview

## 1. Purpose

This document describes the high-level architecture of the self-hosted environment.

It explains the responsibilities of the home network, Windows host, Docker platform, application services and backup system.

The diagram and descriptions are intentionally simplified for public presentation. They are not a complete network map or a deployment specification.

## 2. Architecture at a glance

The environment is organised into five broad areas:

1. Remote access and home networking.
2. Windows host and Docker Desktop.
3. Application routing and containerised services.
4. Application data and supporting databases.
5. Encrypted backup and recovery.

See [current architecture](../diagrams/current-architecture.mmd).

## 3. Host and container platform

The primary host is a Windows 11 system running Docker Desktop.

Docker Desktop provides the container runtime used by the environment.

Ubuntu on WSL 2 is also used as a command-line environment. The Windows Docker CLI and Ubuntu WSL Docker CLI communicate with the same Docker Desktop daemon.

They are therefore two client environments using one container engine, not two independent Docker installations.

This distinction matters when interpreting container listings, resource use and Docker network information.

## 4. Home networking

The UniFi gateway provides local DNS functionality and LAN routing.

These responsibilities are separate from application-level HTTP and HTTPS routing.

The architecture does not publish private IP addressing, Wi-Fi configuration or detailed firewall rules.

## 5. Application routing

Caddy acts as the HTTP/HTTPS reverse proxy.

It handles application-level routing to configured services. This is distinct from the gateway's DNS and LAN-routing responsibilities.

A configured proxy route does not, by itself, prove that the upstream service is currently reachable or healthy.

The documentation therefore distinguishes configured relationships from independently tested connectivity.

## 6. Containerised applications

The environment contains several application and infrastructure categories:

- Photo and video management.
- Databases and application data services.
- Home automation.
- Media streaming.
- Search and web utilities.
- Self-hosted application projects.
- Dashboard and support services.
- Remote-access infrastructure.

The detailed point-in-time inventory is maintained in [container inventory](container-inventory.md).

## 7. Remote access

Two independent remote-access options are documented:

**Twingate**

Twingate is configured to provide access to services across the home network. It is not described as being limited to applications routed through Caddy.

**UniFi Teleport**

UniFi Teleport, accessed through WiFiman, is a separate remote VPN option.

These options are independent. The architecture does not require both to be active simultaneously.

Exact connector configuration, access resources and authentication details are intentionally excluded.

## 8. Data and backup architecture

Application data and configuration are backed up using Restic to an encrypted external repository.

The recorded backup process includes:

- Validation of the external backup destination.
- Validation of the repository and password configuration.
- Validation of backup source paths.
- Validation of a recent database dump.
- Database dump compression-integrity checking.
- Volume Shadow Copy Service support for backup consistency.
- Snapshot verification.
- Automated retention after successful backup verification.

The recorded retention configuration is:

| Retention category | Policy |
|---|---:|
| Daily | 7 |
| Weekly | 4 |
| Monthly | 12 |
| Initial full backup | Protected by tag |

Automatic pruning remains disabled.

Three snapshots were recorded after the retention run on 2 November 2026. This is a historical evidence point, not a claim about the number of snapshots currently stored.

See [backup and recovery](backup-and-recovery.md).

## 9. Recovery architecture

The database recovery work used an isolated PostgreSQL test environment.

The restored database was imported into a temporary container using a dedicated temporary volume. The test environment had no network connection and no published host ports.

The live database was not used as the restore target.

The database import and hash comparison succeeded. Full Immich application recovery has not yet been tested.

## 10. Architecture limitations

The following distinctions are important:

- A configured route is not proof of successful application connectivity.
- A published container port is not proof of public internet exposure.
- A successful database import is not proof of complete application recovery.
- A successful Restic data-pack read is not proof that every application-level recovery scenario works.
- A diagram represents documented relationships, not live monitoring.

## 11. Related documentation

- [Networking](networking.md)
- [Container inventory](container-inventory.md)
- [Security baseline](security-baseline.md)
- [Backup and recovery](backup-and-recovery.md)
- [Known gaps and actions](known-gaps-and-actions.md)

