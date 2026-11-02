# Case Study: Immich Backup Failure, Investigation and Recovery

**Date:** 1–2 November 2026  
**Environment:** Windows 11, Docker Desktop, Immich, Restic 0.19.1 and external Seagate storage  
**Outcome:** Initial backup completed successfully; file-level and PostgreSQL database recovery verified.

## 1. Project objective

The objective was to establish a reliable external backup and recovery process for a self-hosted Immich deployment.

The project involved:

- Creating an encrypted Restic repository.
- Backing up Immich media, database dumps and deployment configuration.
- Integrating Windows Volume Shadow Copy.
- Investigating an interrupted external storage write.
- Completing a successful subsequent backup.
- Checking repository integrity.
- Restoring a database dump from a snapshot.
- Comparing the restored file against the original using SHA-256.
- Importing the recovered database into isolated PostgreSQL.
- Configuring scheduled backup automation and retention.

The investigation was designed to protect the existing Immich deployment. Recovery testing used separate resources rather than modifying the live database.

## 2. Initial architecture

| Component | Configuration |
|---|---|
| Host | Windows 11 |
| Container platform | Docker Desktop |
| Application | Immich |
| Database | PostgreSQL 14 |
| Backup software | Restic 0.19.1 |
| Backup destination | Seagate Expansion Drive |
| Repository | `[LOCAL_BACKUP_REPOSITORY_PATH]` |
| Filesystem | NTFS |
| Repository encryption | Enabled |
| Source snapshot | Windows Volume Shadow Copy |

The repository was successfully initialised. Its ID prefix is `[PRIVATE_RESTIC_REPOSITORY_IDENTIFIER]`.

The repository password was stored separately from the backup data in a protected Windows user directory.

## 3. Initial backup failure

The first full backup attempt failed during writes to the external Seagate drive.

Restic reported:

```text
A device which does not exist was specified
```

Further failures involved temporary repository files. The backup ended with:

```text
Fatal: unable to save snapshot
```

The process returned exit code `1`.

The command had already reported successful creation of a source-side VSS snapshot for `C:\`. However, this was not a completed Restic snapshot.

The distinction was important: successfully creating a source snapshot does not mean that all data has been saved to the destination repository.

## 4. Windows storage investigation

The Windows System event log contained storage-related errors during the failed backup window.

| Time on 1 November 2026 | Provider | Event ID | Observation |
|---|---|---:|---|
| 18:21:20 | Disk | 153 | An I/O operation was retried |
| 18:25:42 | Disk | 51 | Disk paging operation error |
| 18:25:42 | NTFS | 50 | Delayed-write failure involving repository data and filesystem metadata |
| 18:25:44 | NTFS | 98 | Volume subsequently reported healthy |

These events aligned with the Restic write failure.

The evidence established that Windows experienced storage I/O problems during the backup attempt. It did not establish the exact underlying physical cause.

Potential causes considered included:

- USB cable or connector instability.
- Host USB port or controller behaviour.
- External drive enclosure or power behaviour.
- Physical drive reliability.

These remained possible explanations, not confirmed diagnoses.

## 5. Filesystem and device checks

The external drive was subsequently checked using:

```powershell
chkdsk D: /scan
```

The scan reported:

- No filesystem problems.
- Zero bad sectors reported.
- No further filesystem repair required.

Windows later reported the drive as Online and Healthy.

Additional device telemetry included:

| Metric | Reported value |
|---|---:|
| Temperature | 36°C |
| Power-on hours | 657 |
| Total read errors | 184 |
| Uncorrected read errors | 0 |
| Write-error counters | Not reported |

The telemetry did not support a definitive physical diagnosis. The read-error count was recorded as an exposed cumulative counter, not evidence of current unreadable sectors.

## 6. Successful initial backup

A subsequent full Restic backup completed successfully.

The captured output reported:

```text
repository bb1b4b3b opened
no parent snapshot found, will read all files
successfully created snapshot for [c:\]

Files:       48335 new,     0 changed,     0 unmodified
Dirs:        30960 new,     0 changed,     0 unmodified
Added to the repository: 136.272 GiB (135.725 GiB stored)

processed 48335 files, 197.721 GiB in 3:47:13
snapshot 7ec7a616 saved

BACKUP EXIT CODE: 0
BACKUP COMPLETED SUCCESSFULLY
```

### Initial backup results

| Metric | Result |
|---|---:|
| Snapshot ID | `7ec7a616` |
| Files processed | 48,335 |
| Directories processed | 30,960 |
| Data scanned | 197.721 GiB |
| Data added | 136.272 GiB |
| Data stored | 135.725 GiB |
| Duration | 3 hours, 47 minutes, 13 seconds |
| VSS source snapshot | Successful |
| Restic exit code | 0 |

This established a completed initial encrypted backup snapshot.

## 7. Subsequent backup

A second snapshot was successfully created in the existing repository.

| Snapshot | Time | Tags |
|---|---|---|
| `7ec7a616` | 1 November, 18:47:33 | `immich`, `initial-full` |
| `e8d1bd6a` | 1 November, 23:52:05 | `immich`, `daily` |

Both snapshots contained the six expected source paths.

This demonstrated that Restic could reuse the same repository for a subsequent backup.

## 8. Repository integrity check

The following command was executed:

```powershell
restic check
```

Results:

```text
25 / 25 index files loaded
check snapshots, trees and blobs
2 / 2 snapshots
no errors were found
```

Restic also reported 475 additional files that were likely duplicate or unreferenced repository files.

The warning was non-critical. No pruning was performed during the investigation.

The check did not include a complete read of every repository data pack using `--read-data`.

## 9. File-level restoration

A database dump was selected from snapshot `e8d1bd6a`:

`immich-db-backup-20261101T171904-v3.3.5-pg14.19.sql.gz`

Restic successfully located the file within the snapshot and restored it into a separate directory.

### SHA-256 verification

The restored file was checked against the original.

| Metric | Result |
|---|---|
| Original size | 97,168,037 bytes |
| Restored size | 97,168,037 bytes |
| SHA-256 comparison | Identical |

The original and restored files both returned:

```text
0DB89A7CA4DB0474B0051590CE3D3112600EFA2C653120D926192FE77137FF2A
```

PowerShell confirmed:

```text
Identical : True
```

This demonstrated that Restic could retrieve the compressed SQL dump without changing its contents.

## 10. PostgreSQL database recovery

File-level restoration alone was not considered sufficient. The recovered dump was therefore imported into a separate PostgreSQL test container.

The test used the exact image ID of the live Immich PostgreSQL image:

`ghcr.io/immich-app/postgres:14-vectorchord0.4.3-pgvectors0.2.0`

The temporary environment used:

- Container: `immich-db-restore-test`.
- Dedicated Docker named volume.
- Network mode: `none`.
- No published host ports.
- Database: `immich_db`.
- Database user: `immich`.
- Temporary random password.

The import followed the Immich-compatible PostgreSQL restore process, including the search-path transformation.

The database import completed with exit code `0`.

### Database verification

A query against the restored database returned:

```text
immich_db|72|1
```

The results confirmed:

| Check | Result |
|---|---|
| Database name | `immich_db` |
| Base tables | 72 |
| Application schemas | 1 |
| Database import exit code | 0 |

The restored PostgreSQL database also reported eight extensions:

- `cube`
- `earthdistance`
- `pg_trgm`
- `plpgsql`
- `unaccent`
- `uuid-ossp`
- `vchord`
- `vector`

The temporary container and dedicated volume were removed after the test.

The live Immich database was not changed.

### Recovery conclusion

The recovery test demonstrated:

1. Restic could locate and restore the database dump.
2. SHA-256 confirmed byte-for-byte identity with the original.
3. PostgreSQL successfully imported the restored dump.
4. The restored database exposed 72 base tables and the required observed extensions.

A complete Immich application recovery, including starting the application against the recovered database and checking the associated media, remains outstanding.

## 11. Automated backup and retention

A PowerShell script was created:

`Restic-Immich\Backup-Immich.ps1`

Windows Task Scheduler was configured with the task:

`Immich Daily Restic Backup`

The configured daily execution time is 05:00.

The script includes:

- External drive checks.
- Repository validation.
- Password-file checks.
- Source path checks.
- Current-day database dump validation.
- Gzip integrity checking.
- Windows VSS-backed Restic backup.
- Snapshot verification.
- Timestamped logs.
- Automatic retention after successful backup verification.

### Database freshness safeguard

A dry run performed at 00:11 on 2 November correctly refused to run because the latest available dump was from 1 November.

The previous day's dump was:

`immich-db-backup-20261101T171904-v3.3.5-pg14.19.sql.gz`

This was an intentional safety check. The script requires a dump from the current calendar day before it proceeds.

The preceding daily database dumps had generally appeared around 03:36. Therefore the scheduled 05:00 backup was selected to allow time for the daily database dump.

The first successful scheduled run using the latest retention-enabled script is still awaiting confirmation.

### Retention policy

The manually applied retention policy is:

| Snapshot type | Retention |
|---|---:|
| Daily | 7 |
| Weekly | 4 |
| Monthly | 12 |
| Initial full backup | Permanently protected by tag |

The dry-run preview identified both current snapshots for preservation. After applying the policy, both snapshots remained present.

The latest script includes retention after successful backup verification. Automatic pruning remains disabled.

## 12. Technical outcome

The investigation moved from an interrupted external storage write to a completed encrypted backup and a demonstrated PostgreSQL database recovery path.

### Completed achievements

- Initial full encrypted backup.
- Subsequent daily snapshot.
- Repository structural check with no reported errors.
- Database dump located and restored from Restic.
- SHA-256 comparison passed.
- PostgreSQL database import passed.
- 72 base tables and eight extensions confirmed.
- Temporary recovery resources safely removed.
- Daily backup automation configured.
- Snapshot retention policy applied manually.

### Outstanding work

- Confirm the first successful scheduled run using the latest script.
- Complete a full repository data-pack read.
- Monitor the external drive during future backups.
- Test a complete Immich application recovery.
- Validate automated retention under the scheduled task.
- Consider periodic pruning after stable operation and recovery testing.

## 13. Skills demonstrated

This investigation demonstrates hands-on experience in:

- Windows storage troubleshooting.
- Reading and correlating Disk and NTFS events.
- Filesystem checking and device health investigation.
- Docker container administration.
- Encrypted backup repository management.
- Windows Volume Shadow Copy.
- PowerShell automation and error handling.
- PostgreSQL database recovery.
- SHA-256 integrity verification.
- Isolated recovery testing.
- Retention planning.
- Technical incident documentation.

The outcome is a successfully saved backup and a verified database recovery path, with remaining work clearly identified.
