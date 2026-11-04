

# Backup and Recovery

## 1. Purpose

This document describes the backup strategy used for the self-hosted photo and video environment, the Restic automation, recorded backup results and recovery tests.

The objective is to protect application configuration, media and database exports while maintaining a documented recovery path.

The evidence in this document is historical. The most recent specific backup results included here were recorded on 1–2 November 2026.

## 2. Backup design

Restic is used to create encrypted backups in an external repository.

| Component | Recorded configuration |
|---|---|
| Backup software | Restic 0.19.1 |
| Repository type | Encrypted Restic repository |
| Storage destination | External NTFS-formatted storage |
| Backup orchestration | PowerShell |
| Scheduling | Windows Task Scheduler |
| Consistency support | Volume Shadow Copy Service |
| Retention | Automated `forget` policy |
| Physical pruning | Disabled |

The repository password is stored separately from the public documentation. The local password file is protected using Windows access-control permissions.

A separate, recoverable copy of the repository password should be maintained securely. The evidence recorded for this portfolio does not establish that a tested recovery copy is available.

## 3. Backup scope

The recorded backup job covers six source areas:

1. Application environment configuration.
2. Reverse-proxy configuration.
3. Photo and video library content.
4. Certificate material required by the environment.
5. Container deployment configuration.
6. Application media and database backup content.

Sensitive configuration is included only within the encrypted backup process. It must not be copied into this GitHub repository.

The actual local filesystem paths are intentionally omitted from this public document.

## 4. Automation

The PowerShell backup script performs validation before creating a snapshot.

The recorded validation stages include:

- Checking the expected external storage destination.
- Checking storage health and available capacity.
- Checking Restic repository configuration.
- Checking that the repository password is available.
- Checking that all expected source paths exist.
- Checking that a database dump for the expected backup date is available.
- Checking the compressed database dump's integrity.
- Using Volume Shadow Copy Service support.
- Creating the backup snapshot.
- Verifying snapshot details, paths and tags.
- Running retention only after the backup has passed the script's verification steps.

The script and logs are stored outside this public repository.

## 5. Scheduling and retention

The recorded Windows Task Scheduler job runs daily at 05:00 host-local time.

The configured retention policy is:

| Policy | Number retained |
|---|---:|
| Daily | 7 |
| Weekly | 4 |
| Monthly | 12 |
| Initial full backup | Protected by `initial-full` tag |

The script uses Restic retention functionality after a successful backup verification.

Automatic `prune` is deliberately disabled. No claim is made that physical repository compaction has occurred.

## 6. Recorded backup snapshots

The following snapshot records were supplied for the November 2026 backup investigation.

| Snapshot | Recorded date and time | Tags | Description |
|---|---|---|---|
| `7ec7a616` | 1 November 2026, 18:47:33 | `immich`, `initial-full` | Initial full backup |
| `e8d1bd6a` | 1 November 2026, 23:52:05 | `immich`, `daily` | Subsequent daily snapshot |
| `014da08e` | 2 November 2026, 05:00:12 | `immich`, `daily` | Successful scheduled snapshot |

The initial full snapshot recorded:

- 48,335 files.
- 30,960 directories.
- 197.721 GiB scanned.
- 136.272 GiB added.
- 135.725 GiB stored.
- 3 hours, 47 minutes and 13 seconds elapsed.
- Successful VSS operation.
- Exit code 0.

The scheduled run on 2 November 2026 completed successfully.

The subsequent retention operation completed at 05:00:37. The final script exit code was 0, and three snapshots remained after the recorded retention run.

These are operator-confirmed historical results. They have not been independently re-executed as part of this documentation update.

## 7. Repository integrity checks

### Structural repository check

An earlier Restic repository check reported:

- 25 of 25 indexes loaded.
- 2 of 2 snapshots checked.
- No errors found.

The same check reported 475 additional files.

The status and origin of those additional files remain unresolved. They have not been assumed to be safe to delete.

### Full data-pack read

A later full data-read test completed successfully:

| Measure | Result |
|---|---|
| Packs read | 9,221 / 9,221 |
| Elapsed time | 1 hour, 57 minutes and 11 seconds |
| Reported errors | None |

This is positive evidence that the tested repository data packs could be read successfully.

It does not, by itself, establish that every application-level restore scenario works or explain the earlier additional-file observation.

No pruning was performed on the basis of this result.

## 8. Database restore test

A database dump was restored from the recorded Restic backup.

The process included:

1. Selecting the recorded database dump from a backup snapshot.
2. Restoring the file to a separate test location.
3. Comparing the original and restored files using SHA-256.
4. Confirming that the hashes matched.
5. Starting an isolated PostgreSQL test container.
6. Importing the restored database.
7. Checking that the database and expected application tables were present.
8. Removing the temporary test container and volume.

The test used the same PostgreSQL image version as the live deployment at the time of testing.

The temporary test container used a dedicated volume, had no published host ports and was configured without network access.

The import completed successfully with exit code 0. The live database was not modified.

The successful hash comparison establishes that the restored dump matched the original file tested. The successful import establishes that the dump could be imported into the isolated PostgreSQL test environment.

## 9. Recovery boundary

The following recovery stages have recorded successful results:

| Recovery stage | Recorded result |
|---|---|
| Restic backup creation | Successful |
| Scheduled backup execution | Successful |
| Automated retention operation | Successful |
| Full data-pack read | Successful |
| Database dump restoration | Successful |
| Original/restored SHA-256 comparison | Match |
| Isolated PostgreSQL import | Successful |
| Full Immich application recovery | Not tested |

**Complete Immich application recovery remains untested.**

The outstanding work is to bring up the application against a restored database and restored media, then verify application startup, access to media and normal application functionality.

The current evidence should not be described as a complete disaster-recovery test.

## 10. Earlier storage incident

An earlier backup attempt experienced storage-related write errors.

Windows system logs recorded disk and filesystem-related events during the incident, including Disk and NTFS events.

A later `chkdsk` scan reported no filesystem problems and no bad sectors. Subsequent backup operations succeeded.

The physical cause of the original failure was not established.

The incident is documented separately in the [Immich backup incident case study](case-studies/immich-backup-incident.md).

## 11. Remaining limitations

The following items remain open:

- Complete Immich application recovery has not been demonstrated.
- The cause of the earlier storage-related errors remains unconfirmed.
- The 475 additional Restic files remain unexplained.
- Automatic pruning remains disabled.
- A tested independent recovery copy of the repository password is not established by the available evidence.
- Future scheduled runs must be checked rather than assumed successful.

## 12. Operational safety

The following principles apply:

- Do not delete backup snapshots simply because an integrity check reports additional files.
- Do not enable pruning without understanding the repository state and having a verified recovery path.
- Do not test restores against the live database.
- Do not store repository passwords or database dumps in the public repository.
- Do not claim a complete application recovery based only on a successful database import.
- Revalidate external storage identity and health before running backup or retention operations.

