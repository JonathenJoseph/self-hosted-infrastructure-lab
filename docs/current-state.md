# Current State

**Last updated:** 2 November 2026  
**Assessment:** Evidence-based infrastructure audit and backup implementation.

## 1. Host baseline

| Component | Observed state |
|---|---|
| Physical host | HP Elite Mini 800 G9 |
| Processor | Intel Core i9-13900 |
| CPU configuration | 24 cores / 32 logical processors |
| Physical memory | Approximately 31.7 GiB |
| Operating system | Windows 11 |
| Docker Desktop | 4.95.0 |
| Docker Engine | 29.8.2 |
| Docker context | `desktop-linux` |
| WSL version | 3.0.3.0 |
| WSL kernel | 6.18.54.1-1 |
| Linux distribution | Ubuntu on WSL 2 |

The Windows Docker CLI and Ubuntu WSL Docker CLI were observed connecting to the same Docker Desktop daemon.

Ubuntu WSL is a Linux command-line workspace, not a second independent Docker host.

## 2. Application estate

The documented container environment includes:

- Caddy.
- Immich server.
- [Historical per-service storage and persistence details omitted.]
- Immich machine learning.
- Valkey.
- Apprenticeship Tracker web and API.
- Home Assistant.
- SearXNG.
- Jellyfin.
- Homepage.
- Vert.
- Open WebUI.
- Pi-hole.
- Unbound.
- Twingate Connector.
- VEMS and other supporting services.

Detailed images, ports, networks, health checks and storage arrangements are recorded in [Container Inventory](container-inventory.md).

## 3. Network architecture

| Function | Component | Current role |
|---|---|---|
| Local DNS | UniFi Gateway | Local hostname and DNS resolution |
| LAN routing | UniFi Gateway | Routes home-network traffic |
| Reverse proxy | Caddy | HTTP/HTTPS application routing |
| Remote access | Twingate | Operator-confirmed access to any service on the home network |
| Remote VPN | UniFi Teleport through WiFiman | Independent remote-access option |

Twingate and UniFi Teleport are separate access methods.

The local network architecture and Caddy upstreams are documented in [Networking and Routing](networking.md).

## 4. Container and deployment management

Docker Compose project discovery identified the documented projects:

- Apprenticeship Tracker.
- Caddy.
- Homepage.
- Immich.
- Pi-hole.
- VEMS.
- Vert.

Five Windows-hosted Compose definitions were validated during the earlier audit. The referenced Pi-hole Compose definition was missing from the recorded location.

Several containers did not have Compose project labels, so their current reproducible deployment definitions require further investigation.

## 5. Immich backup system

The Immich backup system uses Restic 0.19.1 and an encrypted repository on an external Seagate Expansion Drive.

Repository:

`[LOCAL_BACKUP_REPOSITORY_PATH]`

Repository ID prefix:

`[PRIVATE_RESTIC_REPOSITORY_IDENTIFIER]`

Two successful snapshots were confirmed:

| Snapshot | Timestamp | Tags |
|---|---|---|
| `7ec7a616` | 1 November 2026, 18:47:33 | `immich`, `initial-full` |
| `e8d1bd6a` | 1 November 2026, 23:52:05 | `immich`, `daily` |

The initial full backup processed 48,335 files and 197.721 GiB. It stored 135.725 GiB in the repository.

## 6. Backup recovery verification

The backup has been tested beyond the initial Restic snapshot creation.

| Recovery test | Result |
|---|---|
| Repository structural check | No errors found |
| Database dump discovery | Passed |
| Database dump restoration | Passed |
| Original/restored SHA-256 comparison | Identical |
| PostgreSQL database import | Passed |
| Restored base tables | 72 |
| Application schemas | 1 |
| Reported PostgreSQL extensions | 8 |
| Temporary test environment cleanup | Completed |

The successful PostgreSQL import was performed in an isolated container using the same PostgreSQL image as the live Immich database.

The live database was not modified.

A complete Immich application recovery remains untested.

## 7. Daily backup automation

A PowerShell backup script is stored at:

`[WINDOWS_BACKUP_SCRIPT_PATH]`

Logs are written to:

`[WINDOWS_BACKUP_LOG_DIRECTORY]`

The Windows Task Scheduler task is named:

`Immich Daily Restic Backup`

Configured schedule:

| Setting | Value |
|---|---|
| Frequency | Daily |
| Start time | 05:00 |
| Missed-run handling | Run as soon as possible |
| Concurrent runs | Prevented |
| Script logging | Enabled |

The task was previously observed with:

- State: Ready.
- Last run result: `0`.
- Next run: 2 November 2026 at 05:00.
- Missed runs: 0.

Those results reflect an earlier manual task execution, before the latest retention-enabled script was installed.

The updated script includes a strict requirement for a database dump created on the current calendar day. A dry run at 00:11 on 2 November correctly stopped because the latest available database dump still came from 1 November.

The first successful scheduled execution using the latest script remains outstanding.

## 8. Retention configuration

The following Restic policy was applied manually:

| Recovery point | Retention |
|---|---:|
| Daily | 7 |
| Weekly | 4 |
| Monthly | 12 |
| Initial full snapshot | Protected by `initial-full` tag |

The retention preview and subsequent manual application preserved both existing snapshots.

The updated PowerShell script includes this retention operation after successful backup verification.

Automatic repository pruning remains disabled.

## 9. Previous storage incident

An earlier full backup failed during writes to the external drive.

Windows recorded Disk Event 153, Disk Event 51 and NTFS Event 50 errors during the failure window.

A subsequent filesystem scan reported no NTFS problems or bad sectors. A later Restic backup completed successfully.

The original cause of the external storage interruption remains unresolved and should be monitored during future runs.

## 10. Current status

| Area | Status |
|---|---|
| Windows and Docker infrastructure audit | Documented |
| Local DNS and network architecture | Documented |
| Remote-access architecture | Operator-confirmed and documented |
| Initial Immich backup | Successful |
| Subsequent Restic snapshot | Successful |
| Repository structural check | Passed |
| File restoration | Passed |
| PostgreSQL restoration | Passed |
| Manual snapshot retention | Applied |
| Latest script | Updated |
| Daily Task Scheduler configuration | Ready |
| Latest script first scheduled execution | Awaiting confirmation |
| Full data-pack integrity check | Outstanding |
| Complete Immich recovery drill | Outstanding |
| Automatic pruning | Disabled |

## Related documentation

- [Architecture Overview](architecture-overview.md)
- [Container Inventory](container-inventory.md)
- [Networking and Routing](networking.md)
- [Backup and Recovery](backup-and-recovery.md)
- [Security Baseline](security-baseline.md)
- [Known Gaps and Actions](known-gaps-and-actions.md)
- [Evidence and Method](evidence-and-method.md)
