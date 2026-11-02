# Backup and Recovery

**Last updated:** 2 November 2026  
**Platform:** Windows 11, Docker Desktop, Immich, Restic 0.19.1  
**Status:** Initial backup completed. File restoration and PostgreSQL database restoration successfully tested. Daily automation configured; first scheduled run using the updated script is awaiting confirmation.

## 1. Overview

The Immich environment uses Restic to create encrypted backups on an external Seagate drive.

The backup protects six source paths:

| Source | Purpose |
|---|---|
| `.env` | Immich deployment environment configuration |
| `Caddyfile` | Reverse-proxy configuration |
| `PHOTOS + VIDEOS` | Original photo and video library |
| `caddy-root.crt` | Certificate file |
| `docker-compose.yml` | Container deployment definition |
| `media` | Immich-managed data, including database backup files |

The backup repository is stored at:

`[LOCAL_BACKUP_REPOSITORY_PATH]`

The external drive is an NTFS-formatted Seagate Expansion Drive.

## 2. Restic repository

| Setting | Value |
|---|---|
| Restic version | 0.19.1 |
| Repository ID prefix | `[PRIVATE_RESTIC_REPOSITORY_IDENTIFIER]` |
| Repository format | Version 2 |
| Compression | Auto |
| Encryption | Enabled |
| Backup destination | `[LOCAL_BACKUP_REPOSITORY_PATH]` |
| External filesystem | NTFS |
| Password storage | User-specific LocalAppData password file with restricted ACLs |

The repository password is stored separately from the repository. A recovery copy must be maintained securely outside the computer.

No passwords, secret values or private keys are included in this documentation.

## 3. Successful initial backup

The initial full backup completed successfully on 1 November 2026.

| Metric | Result |
|---|---:|
| Snapshot ID | `7ec7a616` |
| Snapshot tag | `immich, initial-full` |
| Host | `CLOUD-DEVICE` |
| Files processed | 48,335 |
| Directories processed | 30,960 |
| Source data scanned | 197.721 GiB |
| Data added to repository | 136.272 GiB |
| Data stored | 135.725 GiB |
| Duration | 3 hours, 47 minutes, 13 seconds |
| VSS snapshot | Successful |
| Restic exit code | 0 |

Restic successfully created a Windows Volume Shadow Copy snapshot for `C:\` and saved the initial repository snapshot.

## 4. Subsequent daily snapshot

A second snapshot was successfully created on 1 November 2026.

| Metric | Result |
|---|---|
| Snapshot ID | `e8d1bd6a` |
| Timestamp | 1 November 2026, 23:52:05 |
| Tags | `immich, daily` |
| Files processed | 48,335 |
| Source data | 197.721 GiB |
| Restic exit code | 0 |

The second snapshot confirms that Restic can reuse the existing repository and create a subsequent recovery point without creating a separate repository.

Both snapshots were confirmed using `restic snapshots`.

## 5. Repository integrity verification

A repository check was performed using:

`restic check`

Results:

- All 25 index files loaded successfully.
- Both snapshots were checked.
- Snapshot trees and blobs were checked.
- No repository errors were found.
- Restic reported 475 additional files, likely containing duplicate or unreferenced data.

The additional-file warning was non-critical. No pruning was performed as part of this check.

A complete read of every repository data pack using `restic check --read-data` has not yet been recorded.

## 6. File restoration test

A PostgreSQL database dump was selected for an actual restoration test.

**Source file:**

`immich-db-backup-20261101T171904-v3.3.5-pg14.19.sql.gz`

**Snapshot used:** `e8d1bd6a`

Restic successfully located the file within the snapshot and restored it into a separate test directory.

| Check | Result |
|---|---|
| File restored | Yes |
| Original size | 97,168,037 bytes |
| Restored size | 97,168,037 bytes |
| SHA-256 calculated | Yes |
| Original and restored hashes | Identical |

SHA-256:

`0DB89A7CA4DB0474B0051590CE3D3112600EFA2C653120D926FE77137FF2A`

This proves that the restored compressed database dump was byte-for-byte identical to the original.

## 7. PostgreSQL database restoration test

The recovered database dump was imported into a separate temporary PostgreSQL container.

The test used the same PostgreSQL image as the live Immich database:

`ghcr.io/immich-app/postgres:14-vectorchord0.4.3-pgvectors0.2.0`

The temporary environment used:

- Database: `immich_db`
- Database user: `immich`
- A separate Docker named volume
- Docker network mode: `none`
- No published host ports
- A temporary randomly generated password

The restore completed successfully using the Immich-compatible SQL search-path adjustment.

### Restoration results

| Check | Result |
|---|---|
| Temporary PostgreSQL startup | Passed |
| Database dump import | Passed |
| PostgreSQL exit code | 0 |
| Database name | `immich_db` |
| Base tables restored | 72 |
| Application schemas | 1 |
| Extensions present | 8 |

The restored database reported these extensions:

- `cube`
- `earthdistance`
- `pg_trgm`
- `plpgsql`
- `unaccent`
- `uuid-ossp`
- `vchord`
- `vector`

The temporary PostgreSQL container and dedicated test volume were subsequently removed.

The live Immich database was not modified during this test.

**Recovery conclusion:** The encrypted Restic repository has demonstrated successful recovery of a database dump, and the recovered dump has been accepted by an isolated PostgreSQL instance.

A complete Immich application recovery, including starting the application against a restored database and checking the associated media, has not yet been demonstrated.

## 8. Earlier backup incident

An earlier full backup attempt failed while writing to the external Seagate drive.

The failure included:

- Restic temporary-file write errors.
- `A device which does not exist was specified`.
- Windows Disk Event 153.
- Windows Disk Event 51.
- NTFS Event 50 delayed-write failures.

A subsequent `chkdsk D: /scan` reported no filesystem problems and zero bad sectors.

A later backup completed successfully.

The precise cause of the original interruption remains unresolved. The incident and subsequent successful backup are documented in [Case Study: Investigating an Interrupted Immich Backup](case-studies/immich-backup-incident.md).

## 9. Daily backup automation

Windows Task Scheduler is configured with the task:

`Immich Daily Restic Backup`

| Setting | Configuration |
|---|---|
| Schedule | Daily |
| Start time | 05:00 |
| Script | `[WINDOWS_BACKUP_SCRIPT_PATH]` |
| Logging | `[WINDOWS_BACKUP_LOG_DIRECTORY]` |
| Backup method | Restic with Windows VSS |
| Repository | Existing encrypted repository |
| Missed start | Run as soon as possible |
| Overlapping execution | Disabled |

The PowerShell script checks:

1. The Restic executable and password file.
2. The external drive label, filesystem, health and available space.
3. The existing repository can be opened.
4. All six source paths exist.
5. A current-day Immich database dump exists.
6. The database dump passes gzip integrity verification.
7. Restic completes the VSS-backed backup.
8. A new daily snapshot exists and contains all six expected source paths.
9. The configured snapshot retention policy completes.

The updated script has passed a previous full dry run, but its strict current-day database freshness check correctly rejected a dry run performed shortly after midnight before that day's database dump had been generated.

The first successful scheduled execution using the updated script is still awaiting confirmation.

## 10. Retention policy

The following retention policy has been applied manually:

| Recovery point | Retention |
|---|---:|
| Daily | 7 |
| Weekly | 4 |
| Monthly | 12 |
| Initial full backup | Permanently protected by `initial-full` tag |

The retention preview identified both existing snapshots for preservation.

The manual retention operation was then run, and a subsequent snapshot listing confirmed that both snapshots remained present.

The updated PowerShell script also contains the retention operation, which runs only after a successful backup and snapshot verification.

Automatic pruning remains disabled.

### Why pruning is separate

Restic's `forget` command manages snapshot retention. The `prune` command physically removes unreferenced repository data.

Pruning is deliberately excluded from the daily backup script until longer-term operation and repository health have been established.

## 11. Remaining recovery work

| Task | Status |
|---|---|
| Initial full backup | Completed |
| Subsequent daily snapshot | Completed |
| Repository structural check | Passed; 475 additional files reported |
| File-level restoration | Passed |
| SHA-256 comparison | Passed |
| PostgreSQL import test | Passed |
| Temporary test resource cleanup | Completed |
| Daily Task Scheduler configuration | Completed |
| First successful scheduled run with updated script | Awaiting confirmation |
| Automatic retention within scheduled script | Implemented; final operational validation pending |
| Full repository data-read check | Outstanding |
| Complete Immich application recovery drill | Outstanding |
| Long-term external drive reliability observation | Outstanding |
| Automatic pruning | Deliberately disabled |

## 12. Security and recovery considerations

- The Restic repository is encrypted.
- The repository password must be recoverable independently of the Windows installation.
- The `.env` file and certificate are included in the encrypted backup but must never be committed to a public GitHub repository.
- The live PostgreSQL data directory should not be copied as an ordinary file backup while PostgreSQL is running.
- Database dumps and application media are separate recovery requirements.
- The external Seagate drive is connected to the same physical host, so it does not protect against every site-level disaster.
- A successful file restore and PostgreSQL import do not prove that the complete Immich application can be recovered without further testing.

The current backup design provides a working encrypted external recovery point, with additional application-level and long-term reliability testing still planned.
