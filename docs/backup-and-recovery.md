# Backup and Recovery

**Last updated:** 2 November 2026  
**Backup software:** Restic 0.19.1  
**Application:** Immich  
**Repository:** External encrypted Restic repository  
**Status:** Initial backup and database recovery verified. Daily automation configured; first scheduled run using the latest script pending.

## 1. Backup architecture

The Immich environment uses Restic to create encrypted backups on an external Seagate Expansion Drive.

Repository location:

`[LOCAL_BACKUP_REPOSITORY_PATH]`

Filesystem: NTFS.

Repository version: 2.

Compression: Auto.

The password file is stored separately under the Windows user's LocalAppData directory, with restricted access permissions.

The repository ID prefix is `[PRIVATE_RESTIC_REPOSITORY_IDENTIFIER]`.

## 2. Backup source inventory

The existing Restic snapshot contains six source paths:

| Source | Purpose |
|---|---|
| `.env` | Immich deployment environment configuration |
| `Caddyfile` | Reverse-proxy configuration |
| `PHOTOS + VIDEOS` | Original photo and video library |
| `caddy-root.crt` | Certificate file |
| `docker-compose.yml` | Container deployment definition |
| `media` | Immich-managed data, including database backup files |

All six paths are located under the Immich Docker deployment directory.

The `.env` file and certificate are included in the encrypted backup. Their contents must never be published in this repository.

## 3. Initial full backup

The initial full Restic backup completed successfully on 1 November 2026.

| Metric | Result |
|---|---:|
| Snapshot | `7ec7a616` |
| Tags | `immich`, `initial-full` |
| Files processed | 48,335 |
| Directories processed | 30,960 |
| Source data scanned | 197.721 GiB |
| Data added to repository | 136.272 GiB |
| Data stored | 135.725 GiB |
| Duration | 3 hours, 47 minutes, 13 seconds |
| VSS source snapshot | Successful |
| Restic exit code | 0 |

Restic reported:

```text
successfully created snapshot for [c:\]
processed 48335 files, 197.721 GiB in 3:47:13
snapshot 7ec7a616 saved
BACKUP EXIT CODE: 0
BACKUP COMPLETED SUCCESSFULLY
```

The source-side VSS snapshot and the saved Restic snapshot are distinct stages. Both completed successfully in this run.

## 4. Subsequent daily backup

A second snapshot was successfully created in the same repository.

| Metric | Result |
|---|---|
| Snapshot | `e8d1bd6a` |
| Timestamp | 1 November 2026, 23:52:05 |
| Tags | `immich`, `daily` |
| Files processed | 48,335 |
| Source paths | All six expected paths |
| Restic exit code | 0 |

Both snapshots were subsequently confirmed using:

```powershell
restic snapshots
```

No separate repository was created.

## 5. Repository integrity check

The following command was executed:

```powershell
restic check
```

Results:

- All 25 index files loaded.
- Both snapshots were checked.
- Snapshot trees and blobs were checked.
- No errors were found.
- Restic reported 475 additional files, likely containing duplicate or unreferenced repository data.

The additional-file warning was non-critical.

No pruning was performed during the integrity check.

A complete data-pack read using `restic check --read-data` has not yet been recorded. The normal repository check should not be described as a full physical read of every stored data pack.

## 6. File restoration test

A PostgreSQL database dump was selected from the second snapshot:

`immich-db-backup-20261101T171904-v3.3.5-pg14.19.sql.gz`

Snapshot used:

`e8d1bd6a`

Restic located the dump under the Immich media backups directory and restored it into a separate local test folder.

### SHA-256 comparison

| Measurement | Result |
|---|---|
| Original file size | 97,168,037 bytes |
| Restored file size | 97,168,037 bytes |
| Hash algorithm | SHA-256 |
| Comparison | Identical |

Both files returned:

```text
0DB89A7CA4DB0474B0051590CE3D3112600EFA2C653120D926192FE77137FF2A
```

PowerShell returned:

```text
Identical : True
```

This proves that the recovered dump was byte-for-byte identical to the original.

## 7. PostgreSQL database restoration test

The recovered SQL dump was imported into a disposable PostgreSQL container.

The test used the same image as the live Immich PostgreSQL service:

`ghcr.io/immich-app/postgres:14-vectorchord0.4.3-pgvectors0.2.0`

The isolated environment used:

- Database name: `immich_db`
- Database user: `immich`
- Separate Docker named volume.
- Docker network mode: `none`.
- No published host ports.
- A temporary randomly generated password.

The import was performed with the Immich-compatible search-path transformation and PostgreSQL error handling enabled.

The import completed successfully with exit code `0`.

### Restored database inspection

The query returned:

```text
immich_db|72|1
```

This confirmed:

| Database check | Result |
|---|---|
| Database name | `immich_db` |
| Base tables | 72 |
| Application schemas | 1 |
| Import exit code | 0 |

The restored database also reported eight extensions:

- `cube`
- `earthdistance`
- `pg_trgm`
- `plpgsql`
- `unaccent`
- `uuid-ossp`
- `vchord`
- `vector`

The temporary container and dedicated volume were removed after the test.

The live Immich database was not modified.

**Recovery conclusion:** Restic successfully restored the database dump, the restored file matched the original SHA-256 hash, and PostgreSQL accepted the recovered database and exposed its application tables and extensions.

A complete Immich application recovery, including starting the application against a recovered database and checking the associated media, remains outstanding.

## 8. Previous external storage incident

An earlier full backup attempt failed while writing to the Seagate drive.

Restic reported:

```text
A device which does not exist was specified
```

Windows recorded Disk Event 153, Disk Event 51 and NTFS Event 50 errors during the failure window.

A subsequent `chkdsk D: /scan` found no filesystem problems and reported zero bad sectors.

A later full backup completed successfully.

The earlier storage interruption has not been conclusively attributed to the cable, USB port, enclosure or physical drive.

See [Immich Backup Failure and Recovery](case-studies/immich-backup-incident.md).

## 9. Daily PowerShell automation

The daily backup script is stored at:

`[WINDOWS_BACKUP_SCRIPT_PATH]`

Logs are stored under:

`[WINDOWS_BACKUP_LOG_DIRECTORY]`

Windows Task Scheduler contains:

`Immich Daily Restic Backup`

| Setting | Value |
|---|---|
| Schedule | Daily |
| Start time | 05:00 |
| Execution method | Windows PowerShell |
| Backup method | Restic with Windows VSS |
| Repository | Existing encrypted repository |
| Missed start | Run as soon as possible |
| Concurrent execution | Disabled |
| Logging | Timestamped log files |

The script checks:

1. Restic executable availability.
2. Password-file availability.
3. External drive identity, filesystem, health and remaining capacity.
4. Existing repository accessibility.
5. All six source paths.
6. Availability of a database dump created on the current calendar day.
7. Minimum database dump size.
8. Full gzip decompression to check archive integrity.
9. Restic backup completion.
10. The new daily snapshot ID, timestamp, tags and source paths.
11. Retention processing after backup verification.

### Latest script test

A dry-run test on 2 November 2026 at 00:11 failed at the current-day database dump check.

The latest dump available at that time was:

`immich-db-backup-20261101T171904-v3.3.5-pg14.19.sql.gz`

This was the previous day's file. The script correctly refused to proceed.

This was an intentional fail-safe, not a repository or storage failure.

The latest script has not yet completed its first successful scheduled 05:00 run.

## 10. Retention policy

The retention policy has been previewed and applied manually.

| Snapshot category | Retention |
|---|---:|
| Daily | 7 |
| Weekly | 4 |
| Monthly | 12 |
| Initial full backup | Protected with `initial-full` tag |

The dry-run preview identified both existing snapshots for preservation.

The retention operation was then applied manually, and `restic snapshots` subsequently confirmed both snapshots remained present.

The updated PowerShell script includes automatic retention after a successful new backup and snapshot verification.

Automatic pruning remains disabled.

### Why pruning is separate

Restic `forget` manages which snapshots are retained.

Restic `prune` physically removes repository data that is no longer needed by retained snapshots.

Pruning is intentionally not included in the daily task at this stage.

## 11. Remaining recovery and operational work

| Task | Status |
|---|---|
| Initial full backup | Complete |
| Second daily snapshot | Complete |
| Repository structural check | Passed |
| SHA-256 file restore comparison | Passed |
| PostgreSQL database import | Passed |
| Restored database table inspection | Passed |
| Temporary recovery resources | Removed |
| Manual retention policy | Applied |
| Task Scheduler configuration | Complete |
| Latest script dry-run with a current-day database dump | Pending |
| First successful scheduled run using latest script | Pending |
| Full data-pack integrity check | Outstanding |
| Complete Immich application recovery test | Outstanding |
| Long-term external storage reliability | Monitoring required |
| Automatic pruning | Deliberately disabled |

## 12. Security and recovery limitations

- The Restic repository is encrypted.
- The repository password must be recoverable independently of the Windows host.
- The `.env` file and certificate are included in the encrypted repository and must not be committed to GitHub.
- The live PostgreSQL data directory should not be copied as an ordinary file backup while PostgreSQL is running.
- The external Seagate drive is connected to the same physical host. It is not an off-site backup.
- A successful backup and database import do not prove that the complete Immich application can be recovered without further testing.
- The external storage interruption remains an unresolved reliability concern.

## Related documentation

- [Immich Backup Failure and Recovery](case-studies/immich-backup-incident.md)
- [Current State](current-state.md)
- [Known Gaps and Actions](known-gaps-and-actions.md)
- [Evidence and Method](evidence-and-method.md)
