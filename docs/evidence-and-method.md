# Evidence and Method

**Last updated:** 2 November 2026

## 1. Purpose

This document describes how infrastructure findings are collected, interpreted and recorded.

The portfolio separates direct technical evidence from operator confirmation and assumptions.

The purpose is to make the documentation reproducible and prevent untested assumptions from being presented as established facts.

## 2. Evidence classifications

| Classification | Meaning |
|---|---|
| Verified | Directly supported by command output, configuration inspection or successful test result |
| User-confirmed | Explicitly confirmed by the infrastructure operator |
| Inferred | Reasonable conclusion based on available evidence, but not demonstrated directly |
| Open | Further investigation or validation is required |
| Deferred | Intentionally outside the current investigation scope |

## 3. Evidence collection methods

| Area | Evidence gathered |
|---|---|
| Windows host | Windows build, system information, CPU, memory and storage |
| WSL | WSL version, kernel and distribution status |
| Docker | `docker version`, `docker ps -a`, `docker inspect`, `docker stats` |
| Docker networking | `docker network ls` and `docker network inspect` |
| Docker storage | `docker volume ls`, `docker volume inspect`, `docker system df -v` |
| Compose | `docker compose ls` and `docker compose config` |
| Caddy | Caddyfile, site snippets, `caddy validate` and container logs |
| Firewall | Windows firewall profile, rule, port and application inspection |
| Storage | Directory sizes, container mounts and volume inspection |
| Database dumps | Gzip integrity checks and file metadata |
| Restic repository | Repository initialisation, snapshot listing, `restic check`, `restic find` and `restic restore` |
| Backup automation | PowerShell dry runs, script logs and Task Scheduler status |
| Storage incident | Disk and NTFS Windows event logs, `chkdsk D: /scan` and exposed device telemetry |
| PostgreSQL recovery | Isolated container import and database inspection |
| Remote access | Operator confirmation of Twingate access to any service on the home network |

## 4. Initial Restic backup evidence

The initial full backup output reported:

- Repository opened successfully.
- VSS snapshot of `C:\` created successfully.
- 48,335 files processed.
- 30,960 directories processed.
- 197.721 GiB scanned.
- 136.272 GiB added to the repository.
- 135.725 GiB stored.
- Duration of 3 hours, 47 minutes and 13 seconds.
- Snapshot `7ec7a616` saved.
- Exit code `0`.

This establishes that an initial Restic snapshot was successfully saved.

## 5. Subsequent snapshot evidence

A second snapshot was created:

| Snapshot | Time | Tags |
|---|---|---|
| `7ec7a616` | 1 November 2026, 18:47:33 | `immich`, `initial-full` |
| `e8d1bd6a` | 1 November 2026, 23:52:05 | `immich`, `daily` |

A subsequent `restic snapshots` listing confirmed both snapshots and their six source paths.

## 6. Repository integrity evidence

The command:

```powershell
restic check
```

reported:

```text
25 / 25 index files loaded
2 / 2 snapshots
no errors were found
```

Restic also reported 475 additional files, likely containing duplicate or unreferenced data.

The check did not include `--read-data`, so the portfolio does not claim that every repository data pack has been fully read and verified.

## 7. File restoration evidence

The database dump:

`immich-db-backup-20261101T171904-v3.3.5-pg14.19.sql.gz`

was located in snapshot `e8d1bd6a` and restored to a separate test directory.

The original and restored files both measured 97,168,037 bytes.

PowerShell returned identical SHA-256 results:

```text
0DB89A7CA4DB0474B0051590CE3D3112600EFA2C653120D926192FE77137FF2A
```

The comparison returned:

```text
Identical : True
```

This supports the claim that the recovered database dump was byte-for-byte identical to the original.

## 8. PostgreSQL recovery evidence

The restored dump was imported into a temporary isolated PostgreSQL container using the same image as the live Immich database.

The test environment used:

- PostgreSQL 14.
- Database name `immich_db`.
- Database user `immich`.
- A dedicated Docker volume.
- Network mode `none`.
- No published ports.
- A temporary generated password.

The import completed with exit code `0`.

The subsequent inspection query returned:

```text
immich_db|72|1
```

This confirmed:

- Database name: `immich_db`.
- Base tables: 72.
- Application schemas: 1.

The restored database reported the following extensions:

```text
cube
earthdistance
pg_trgm
plpgsql
unaccent
uuid-ossp
vchord
vector
```

The temporary container and volume were removed after testing.

The live Immich database was not modified.

## 9. Backup automation evidence

The daily Restic script is stored at:

`Restic-Immich\Backup-Immich.ps1`

The Windows Task Scheduler task is:

`Immich Daily Restic Backup`

The configured trigger is daily at 05:00.

The task was previously observed with:

- State `Ready`.
- Last task result `0`.
- Next run `2 November 2026, 05:00`.
- No missed runs.

Those observations refer to the earlier task execution and script version.

The latest script introduced a strict current-day database dump requirement. A dry run at 00:11 on 2 November stopped because the most recent database dump was still dated 1 November.

This confirms the freshness check prevented a backup from proceeding with a previous-day database dump.

The first successful scheduled execution of the latest script remains unverified.

## 10. Retention evidence

A Restic retention preview used:

```powershell
restic forget `
    --tag immich `
    --keep-daily 7 `
    --keep-weekly 4 `
    --keep-monthly 12 `
    --keep-tag initial-full `
    --dry-run
```

The preview preserved both existing snapshots.

The retention command was subsequently run without `--dry-run`. A later snapshot listing confirmed that both existing snapshots remained present.

The updated daily script contains the retention policy after successful backup verification.

Automatic pruning is not enabled.

## 11. Remote access evidence

The operator confirmed that Twingate allows access to any service on the home network.

The Twingate Connector was observed using Docker host networking.

UniFi Teleport is documented as a separate remote VPN option through WiFiman.

The documentation treats the Twingate capability as user-confirmed configuration information rather than an independently captured off-site network test.

## 12. Important limitations

- All observations are point-in-time and may change as the infrastructure is updated.
- The initial backup and PostgreSQL restore were successfully tested, but a complete Immich application recovery was not performed.
- The normal Restic repository check did not read every repository data pack.
- The earlier external storage interruption remains unexplained.
- The first successful scheduled execution using the latest backup script has not been confirmed.
- The long-term behaviour of the external drive has not been established.
- Published Docker ports do not, on their own, prove internet exposure.
- Docker network membership does not prove application-level communication.
- A healthy container status does not necessarily prove end-to-end application health.
- The portfolio does not contain credentials, API keys, private certificate material or raw database dumps.

## 13. Reproducibility and documentation maintenance

When refreshing the audit:

1. Capture timestamped command output.
2. Record the command and the purpose of each test.
3. Redact credentials, hostnames and sensitive paths.
4. Separate observed configuration from operator confirmation.
5. Record failures as well as successful outcomes.
6. Keep raw private evidence outside the public portfolio.
7. Update the snapshot date when refreshing point-in-time results.

## 14. Change history

| Date | Change |
|---|---|
| 1 November 2026 | Initial infrastructure documentation baseline |
| 1 November 2026 | Initial successful Restic snapshot `7ec7a616` |
| 1 November 2026 | Subsequent snapshot `e8d1bd6a` |
| 1 November 2026 | Repository structural check completed |
| 1 November 2026 | Database dump restored and SHA-256 comparison passed |
| 1 November 2026 | Isolated PostgreSQL database import and inspection passed |
| 1 November 2026 | Manual retention policy applied |
| 2 November 2026 | Updated backup automation and retention documentation |
| 2 November 2026 | Twingate remote-access description updated following operator confirmation |
