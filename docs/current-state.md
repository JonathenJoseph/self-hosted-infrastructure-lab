

# Current State and Recorded Evidence

## 1. Purpose

This document summarises the recorded state of the self-hosted infrastructure and distinguishes confirmed capabilities from outstanding work.

It is a point-in-time portfolio record. It is not a live monitoring page.

The backup-specific results below reflect evidence recorded on 1–2 November 2026. Other infrastructure details are based on the environment review and may have changed since their observation.

## 2. Host platform

| Component | Recorded state |
|---|---|
| Host operating system | Windows 11 |
| Container platform | Docker Desktop |
| Container engine | Docker Engine, Linux/amd64 |
| Linux command-line environment | Ubuntu on WSL 2 |
| Docker CLI relationship | Windows and WSL clients use the same Docker Desktop daemon |

The WSL environment is not a separate Docker daemon in this setup.

## 3. Main service groups

The recorded container estate includes:

- Reverse proxying.
- Photo and video management.
- PostgreSQL and supporting data services.
- Home automation.
- Media streaming.
- Self-hosted application projects.
- DNS-related services.
- Dashboard and utility applications.
- Remote-access infrastructure.

See [container inventory](container-inventory.md) for the service-level summary.

## 4. Network architecture

The documented separation of responsibilities is:

| Component | Responsibility |
|---|---|
| UniFi gateway | Local DNS and LAN routing |
| Caddy | HTTP/HTTPS reverse proxy |
| Twingate | Remote access to services across the home network |
| UniFi Teleport | Independent remote VPN option |
| Docker | Application and service containerisation |

Twingate is not described as restricted to Caddy-hosted services.

UniFi Teleport is a separate remote-access option. Neither service should be assumed to depend on the other.

## 5. Backup state recorded on 2 November 2026

The following results were supplied from the backup investigation.

| Item | Recorded result |
|---|---|
| Backup software | Restic 0.19.1 |
| Scheduled backup | Completed successfully |
| Third recorded snapshot | `014da08e` |
| Snapshot time | 2 November 2026, 05:00:12 |
| Snapshot tags | `immich`, `daily` |
| Retention operation | Completed at 05:00:37 |
| Final script exit code | 0 |
| Snapshots after retention | 3 |
| Full data-pack read | 9,221 / 9,221 packs |
| Reported data-pack read errors | None |
| Automatic pruning | Disabled |

These results are historical and operator-confirmed. They are not confirmation that every subsequent scheduled run succeeded.

## 6. Database recovery state

A database dump was restored from the recorded backup.

The original and restored dump hashes matched using SHA-256.

The restored dump was successfully imported into a temporary isolated PostgreSQL test environment.

The test environment was separate from the live database and was removed after testing.

This establishes successful database dump restoration and import under the tested conditions.

It does not establish full Immich application recovery.

## 7. Storage incident

An earlier backup attempt experienced storage-related write errors.

A later filesystem scan reported no filesystem problems and no bad sectors. Subsequent backup operations completed successfully.

The physical cause of the initial issue remains unconfirmed.

## 8. Outstanding items

The following are not represented as completed:

- Full Immich application recovery using restored database and media.
- Explanation of the 475 additional files reported by an earlier Restic check.
- Automatic Restic pruning.
- Confirmation of a tested independent repository-password recovery copy.
- Establishment of the physical cause of the earlier storage errors.
- Revalidation of all container security and persistence observations.
- Complete revalidation of every configured reverse-proxy upstream.

## 9. Interpretation

The strongest verified recovery evidence is the successful database dump restoration, SHA-256 comparison, isolated PostgreSQL import and full Restic data-pack read.

These results are meaningful, but they should not be used to claim that the complete Immich application has passed a disaster-recovery test.

