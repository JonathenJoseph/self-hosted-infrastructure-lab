

# Case Study: Immich Backup and Recovery Investigation

## 1. Overview

This case study documents an investigation into backup failures affecting a self-hosted Immich environment.

The work covered storage troubleshooting, encrypted backup creation, scheduled automation, retention, repository integrity checking and isolated database recovery.

The most important outcome was not simply producing a successful backup. It was building a repeatable process and testing whether backed-up database data could be restored.

This case study is based on operator-provided records and test results. The results were not independently re-executed while preparing this public document.

## 2. Initial problem

An earlier backup attempt experienced storage-related write errors while writing to external backup storage.

Windows system logs recorded disk and filesystem-related events during the incident, including:

- Disk-related events.
- Storage I/O-related events.
- NTFS-related events.

The initial failure prevented the backup operation from completing successfully.

The investigation needed to establish whether the issue was caused by an obvious filesystem problem, whether subsequent backups could succeed and whether the resulting data could be read and restored.

## 3. Initial investigation

A later filesystem scan completed without reporting filesystem problems or bad sectors.

This was useful evidence, but it did not establish the physical cause of the earlier failure.

Subsequent backup attempts completed successfully.

The incident was therefore recorded as an unresolved storage-related failure rather than a confirmed hardware fault.

## 4. Backup implementation

Restic was used to create encrypted backups on external storage.

The backup process was automated through PowerShell and Windows Task Scheduler.

The script was designed to validate its inputs before creating a snapshot.

Recorded checks included:

- External destination validation.
- Storage health and free-space checks.
- Repository configuration.
- Password availability.
- Source-path availability.
- Database dump availability.
- Compressed dump integrity.
- Volume Shadow Copy Service support.
- Snapshot creation and verification.
- Retention after successful backup verification.

The exact local paths and repository identifiers are omitted from this public case study.

## 5. Backup evidence

The initial full snapshot was recorded on 1 November 2026.

Its recorded results included:

| Metric | Result |
|---|---:|
| Files | 48,335 |
| Directories | 30,960 |
| Data scanned | 197.721 GiB |
| Data added | 136.272 GiB |
| Data stored | 135.725 GiB |
| Duration | 3 hours, 47 minutes, 13 seconds |
| VSS | Successful |
| Script exit code | 0 |

Two further snapshots were recorded before the successful scheduled backup on 2 November.

### Successful scheduled run

On 2 November 2026, a scheduled backup completed successfully.

| Result | Value |
|---|---|
| Snapshot | `014da08e` |
| Snapshot time | 05:00:12 |
| Tags | `immich`, `daily` |
| Retention completed | 05:00:37 |
| Final script exit code | 0 |
| Snapshots remaining | 3 |

The results were supplied by the operator. They were not independently re-run during preparation of this portfolio document.

## 6. Retention

The recorded retention policy was:

| Category | Retention |
|---|---:|
| Daily | 7 |
| Weekly | 4 |
| Monthly | 12 |
| Initial full snapshot | Protected by tag |

Retention was configured to run after the backup had passed its verification steps.

Automatic pruning remained disabled.

This distinction matters: retention selection and physical repository pruning are separate operations. The documentation does not claim that pruning or compaction occurred.

## 7. Repository integrity

An earlier repository check loaded all 25 indexes and checked both snapshots then present. It reported no errors, but also reported 475 additional files.

The status of those files remains unresolved.

A separate full data-pack read subsequently completed:

| Metric | Result |
|---|---:|
| Packs read | 9,221 / 9,221 |
| Duration | 1 hour, 57 minutes, 11 seconds |
| Reported errors | None |

This is positive evidence that the repository's tested data packs could be read.

It does not establish what the additional files represent, and it does not prove that every possible application recovery scenario will work.

No pruning was performed on the basis of the additional-file report.

## 8. Database restoration

A database dump was restored from a recorded Restic snapshot.

The original database dump and restored copy were compared using SHA-256.

The hashes matched.

The restored dump was then imported into an isolated PostgreSQL test environment.

The test used:

- The same PostgreSQL image version as the live deployment at the time.
- A dedicated temporary data volume.
- No published host ports.
- No network connectivity.
- A separate test database.
- Temporary credentials.

The import completed successfully with exit code 0.

The database was queried to confirm that the expected database and application table structure were present.

The temporary container and volume were subsequently removed.

The live database was not modified by the test.

## 9. What the investigation demonstrated

The recorded evidence supports the following conclusions:

1. The initial external-storage backup failure did not prevent later successful backup operations.
2. A full Restic backup could be created successfully.
3. The scheduled backup completed successfully on 2 November 2026.
4. The configured retention operation completed successfully after the backup.
5. All 9,221 packs in the recorded full data-read test were read without reported errors.
6. A database dump could be restored from the backup.
7. The restored database dump matched the original using SHA-256.
8. The restored dump could be imported into an isolated PostgreSQL test environment.

These are substantial recovery milestones, but they have a defined scope.

## 10. What remains unproven

Complete Immich application recovery has not been tested.

The outstanding application-level recovery work is to:

- Restore the required database and media data into a separate environment.
- Start Immich against the restored database.
- Verify that the application starts correctly.
- Confirm that media is accessible.
- Confirm that expected application functionality works.
- Document any recovery-specific configuration changes.

Until those steps are completed, this case study must not claim complete Immich disaster recovery.

The physical cause of the earlier storage incident also remains unresolved.

The 475 additional Restic files remain unexplained, and automatic pruning remains disabled.

## 11. Lessons learned

### Validate backup inputs before writing

Checking storage, source paths, repository availability and database dump integrity helps prevent an incomplete backup from being treated as successful.

### A successful backup is not enough

A backup should be tested through integrity checks and actual restoration.

### Isolate recovery testing

Using a separate PostgreSQL test container and volume prevented the restore test from modifying the live database.

### Hashes provide useful evidence

Matching SHA-256 hashes demonstrated that the restored database dump matched the original file used in the test.

### Keep conclusions proportional to the evidence

A successful database restore is valuable, but it is not the same as a complete application recovery.

### Avoid destructive repository maintenance without understanding the result

The 475 additional files were left unresolved rather than removed through speculative pruning.

## 12. Final outcome

The investigation progressed from an unsuccessful storage-related backup attempt to a functioning scheduled backup and a successfully tested database restore.

It demonstrates practical experience in:

- Troubleshooting.
- Windows system investigation.
- Backup automation.
- Encrypted storage.
- Retention management.
- Repository integrity checking.
- Database recovery.
- Test isolation.
- Evidence-based technical reporting.

The remaining work is documented explicitly rather than presented as completed.

