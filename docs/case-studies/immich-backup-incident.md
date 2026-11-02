# Case Study: Immich Backup Failure, Investigation and Recovery

**Date:** 1–2 November 2026  
**Environment:** Windows 11, Docker Desktop, Immich, Restic 0.19.1 and external Seagate storage  
**Status:** Initial backup and database recovery tests successful. Daily automation configured; continued operational verification pending.

## 1. Project overview

The objective was to establish a reliable external backup and recovery process for a self-hosted Immich installation.

The environment stores approximately 197.7 GiB of source data across original photos and videos, Immich-managed media, database backup files and deployment configuration.

The project involved:

- Establishing an encrypted Restic repository on external storage.
- Creating an initial full backup using Windows Volume Shadow Copy.
- Investigating an interrupted backup caused by external storage write errors.
- Completing a subsequent successful backup.
- Verifying repository integrity.
- Restoring an individual database dump from a snapshot.
- Comparing original and restored file hashes.
- Importing the recovered database into an isolated PostgreSQL environment.
- Configuring daily backup automation and snapshot retention.

The investigation was designed to protect the existing Immich installation. Database recovery tests were performed separately without modifying the live database.

## 2. Initial backup architecture

| Component | Configuration |
|---|---|
| Source application | Immich |
| Host operating system | Windows 11 |
| Container platform | Docker Desktop |
| Linux database | PostgreSQL 14 |
| Backup application | Restic 0.19.1 |
| Repository destination | `[LOCAL_BACKUP_REPOSITORY_PATH]` |
| External storage | Seagate Expansion Drive |
| Filesystem | NTFS |
| Repository encryption | Enabled |
| Source snapshot mechanism | Windows Volume Shadow Copy |

The existing Restic repository was initialised successfully. Its ID prefix is `[PRIVATE_RESTIC_REPOSITORY_IDENTIFIER]`.

The password is stored separately in the Windows user's LocalAppData directory, with restricted filesystem permissions.

## 3. Initial backup failure

The first full Restic backup attempt failed during writes to the external drive.

Restic reported errors including:

```text
A device which does not exist was specified

unable to save snapshot

exit code 1
```

The failure occurred after the source-side Windows Volume Shadow Copy had been created successfully.

This distinction was important: creating the VSS snapshot did not mean that Restic had successfully committed the backup to its repository.

### Windows event-log investigation

The Windows System event log contained storage-related events during the failed backup window.

| Time | Provider | Event ID | Observation |
|---|---|---:|---|
| 18:21:20 | Disk | 153 | An I/O operation was retried |
| 18:25:42 | Disk | 51 | Disk paging operation error |
| 18:25:42 | NTFS | 50 | Delayed write failure involving repository data and filesystem metadata |
| 18:25:44 | NTFS | 98 | Volume later reported healthy |

The timing of the disk and NTFS errors aligned with the Restic write failures.

This established that Windows had experienced storage I/O problems during the backup attempt.

It did not establish the exact physical cause.

Potential causes considered included:

- USB cable or connector instability.
- Host USB port or controller behaviour.
- External drive enclosure or power behaviour.
- A problem with the physical drive.

These remained possibilities rather than confirmed diagnoses.

## 4. Storage health investigation

The external drive was subsequently inspected using Windows storage tools.

A filesystem scan was performed:

```powershell
chkdsk D: /scan
```

The scan reported:

- No filesystem problems.
- Zero bad sectors reported by the scan.
- No further filesystem action required.

Windows subsequently reported the external drive as Online and Healthy.

Additional device information included:

| Metric | Observed result |
|---|---:|
| Temperature | 36°C |
| Power-on hours | 657 |
| Total read errors | 184 |
| Uncorrected read errors | 0 |
| Write-error counters | Not reported |

These figures were not sufficient to diagnose the physical drive. The read-error count was recorded as an exposed cumulative counter, not proof of current unreadable sectors.

The external volume later passed its health checks, allowing a subsequent backup attempt.

## 5. Successful initial backup

A later full backup completed successfully.

Restic reported:

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
| Source data scanned | 197.721 GiB |
| Data added to repository | 136.272 GiB |
| Data stored | 135.725 GiB |
| Duration | 3 hours, 47 minutes, 13 seconds |
| VSS snapshot | Successful |
| Restic exit code | 0 |

This established a completed initial Restic snapshot.

## 6. Subsequent backup and repository verification

A second snapshot was created using the existing repository.

| Snapshot | Timestamp | Tags |
|---|---|---|
| `7ec7a616` | 1 November 2026, 18:47:33 | `immich`, `initial-full` |
| `e8d1bd6a` | 1 November 2026, 23:52:05 | `immich`, `daily` |

Both snapshots included the six expected source paths.

A repository check was then executed:

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

Restic also reported 475 additional files that were likely duplicate or unreferenced repository data. This warning was non-critical, and automatic pruning was not performed during the investigation.

## 7. Restoring a database backup from Restic

The next objective was to prove that a database backup could actually be retrieved from the repository.

The database dump selected was:

`immich-db-backup-20261101T171904-v3.3.5-pg14.19.sql.gz`

It was located inside snapshot `e8d1bd6a` using:

```powershell
restic find --snapshot e8d1bd6a "immich-db-backup-20261101T171904-v3.3.5-pg14.19.sql.gz"
```

Restic identified the file under:

```text
/C/Users/jonat/Immich_docker/media/backups/
```

The file was then restored into a separate local recovery-test directory.

### File restoration results

| Check | Result |
|---|---|
| File located in snapshot | Passed |
| File restored | Passed |
| Original file size | 97,168,037 bytes |
| Restored file size | 97,168,037 bytes |
| SHA-256 verification | Passed |

Original and restored SHA-256:

```text
0DB89A7CA4DB0474B0051590CE3D3112600EFA2C653120D926192FE77137FF2A
```

The comparison returned:

```text
Identical : True
```

This demonstrated that Restic could retrieve the database dump without altering its contents.

## 8. Isolated PostgreSQL database recovery

File-level restoration alone was not considered sufficient evidence of database recoverability.

An isolated PostgreSQL test environment was therefore created using the same image as the live Immich database:

```text
ghcr.io/immich-app/postgres:14-vectorchord0.4.3-pgvectors0.2.0
```

The test environment used:

- Container: `immich-db-restore-test`
- Separate Docker named volume: `immich-db-restore-test-data`
- Network mode: `none`
- No published host ports
- Database: `immich_db`
- User: `immich`
- Temporary randomly generated password

The recovered SQL dump was imported using the Immich-compatible PostgreSQL restore process, including the required search-path adjustment.

The import completed successfully with exit code `0`.

### Database verification

A subsequent PostgreSQL query reported:

```text
immich_db|72|1
```

This confirmed:

| Check | Result |
|---|---|
| Database name | `immich_db` |
| Base tables | 72 |
| Application schemas | 1 |
| PostgreSQL import exit code | 0 |

The restored database also reported eight extensions:

- `cube`
- `earthdistance`
- `pg_trgm`
- `plpgsql`
- `unaccent`
- `uuid-ossp`
- `vchord`
- `vector`

The temporary container and dedicated volume were subsequently removed.

The live Immich database remained untouched.

### Recovery conclusion

The investigation successfully demonstrated three levels of recovery:

1. Restic could locate and restore the database dump.
2. The restored file was byte-for-byte identical to the original.
3. PostgreSQL accepted the recovered dump and exposed the restored database tables and extensions.

This is stronger evidence than simply reporting that a backup job completed.

A complete Immich application recovery, including starting the application against a restored database and confirming its associated media, remains untested.

## 9. Daily backup automation

A PowerShell script was developed to consolidate the backup process:

`[WINDOWS_BACKUP_SCRIPT_PATH]`

The script includes:

- External drive identity and health checks.
- Repository accessibility checks.
- Password-file availability checks.
- Validation of all six original backup paths.
- Current-day Immich database dump detection.
- Gzip integrity verification.
- Windows VSS-backed Restic backup.
- Snapshot verification.
- Timestamped logs.
- Failure exit codes.
- Automatic snapshot retention after successful backup verification.

Windows Task Scheduler was configured with:

| Setting | Value |
|---|---|
| Task name | `Immich Daily Restic Backup` |
| Schedule | Daily |
| Execution time | 05:00 |
| Execution account | Windows user account |
| Elevated execution | Enabled |
| Missed-run handling | Run as soon as possible |
| Overlapping executions | Disabled |

The script's dry-run successfully verified the repository, source paths and database archive before the strict current-day check was introduced.

A later dry run just after midnight correctly failed because the previous day's database dump was still the latest available dump. No backup snapshot was created by that dry run.

The scheduled task had previously reported a successful manual execution, but the first successful scheduled run using the updated retention-enabled script remains to be confirmed.

## 10. Snapshot retention

A retention policy was previewed and subsequently applied manually.

| Recovery point | Retention |
|---|---:|
| Daily | 7 |
| Weekly | 4 |
| Monthly | 12 |
| Initial full snapshot | Permanently protected by `initial-full` |

The dry run identified both existing snapshots for preservation.

After applying the policy, `restic snapshots` confirmed that both snapshots remained available.

The updated PowerShell script also includes this policy, running only after a successful backup and snapshot verification.

Automatic pruning remains disabled.

This separates snapshot selection from physical repository cleanup and protects the existing recovery history while the daily process is established.

## 11. Final assessment

The investigation moved from an interrupted storage write to a successful encrypted backup and a demonstrated PostgreSQL database restoration.

### Verified achievements

- Initial full Restic backup completed successfully.
- A subsequent daily snapshot was created.
- Repository structural check reported no errors.
- Database dump restored from the repository.
- SHA-256 hashes matched exactly.
- PostgreSQL accepted the recovered database.
- 72 base tables and eight extensions were confirmed.
- Temporary recovery resources were cleaned up safely.
- Daily Task Scheduler configuration was created.
- Snapshot retention policy was applied without deleting either existing snapshot.

### Outstanding work

- Confirm the first successful scheduled run using the latest script.
- Observe subsequent backup runs for external storage reliability.
- Perform a full repository data-read integrity check.
- Test a complete Immich application recovery, including media access.
- Validate that the daily retention operation works under the scheduled task.
- Establish and validate a routine for repository pruning.

## 12. Skills demonstrated

This case study demonstrates practical experience in:

- Windows system troubleshooting.
- Reading and correlating Disk and NTFS event logs.
- External storage and filesystem diagnostics.
- Linux container administration through Docker Desktop.
- Encrypted backup repository management.
- Windows Volume Shadow Copy integration.
- PowerShell automation and error handling.
- PostgreSQL database restoration.
- Hash-based file integrity verification.
- Recovery testing in an isolated container.
- Evidence-based technical documentation.
- Separating successful backup completion from proven application recovery.

The outcome is a functioning initial backup and a verified database recovery path, with further operational testing clearly identified.
