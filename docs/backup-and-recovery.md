# Backup and Recovery

**Snapshot date:** 1 November 2026  
**Status:** Initial full Restic backup completed successfully. Post-backup repository verification and restore testing remain outstanding.

## Immich data locations and approximate size

| Data set | Approximate size | Observed content | Backup handling |
|---|---:|---|---|
| Original photos and videos | 129.83 GiB | About 19,576 files | Included in the configured file-level backup |
| Immich media directory | 67.89 GiB | About 28,755 files, including application-managed media and PostgreSQL dumps | Included in the configured backup |
| PostgreSQL live data directory | 0.55 GiB | Live database files | Do not copy as an ordinary live directory; use database dumps or a consistent database-aware backup |
| Separate storage directory | Empty at inspection | No files found | Re-check before assuming it remains unused |

These measurements are point-in-time estimates, not capacity guarantees.

## PostgreSQL dump verification

A manual Immich database dump was created on 1 November 2026. The gzip file was approximately 97 MB. It was decompressed successfully to approximately 238 MiB, and gzip integrity passed.

This verifies archive-stream integrity only. It does **not** prove that the dump can be restored by PostgreSQL or that the restored Immich application starts correctly. A database restore test remains outstanding.

Fourteen daily compressed database dump files were observed in the Immich backups directory during the initial inspection. Retention and automatic dump generation should be confirmed.

## Restic repository preparation

| Item | Observed state |
|---|---|
| Restic | Version 0.19.1 |
| External target | Seagate USB drive, mounted as `D:` |
| Repository | `[LOCAL_BACKUP_REPOSITORY_PATH]` |
| Repository ID prefix | `[PRIVATE_RESTIC_REPOSITORY_IDENTIFIER]` |
| Repository initialisation | Successful |
| Password file | Created under the current Windows user's LocalAppData Restic directory and ACL-restricted |
| VSS dry run | Successful |
| Initial full backup | **Completed successfully** |
| Snapshot ID | `7ec7a616` |
| Restic exit code | `0` |
| Restore test | Not yet performed |

The repository is encrypted. The password must be recoverable independently of the Windows installation. Keep a recovery copy in a password manager or other secure off-device location. Never commit the password file or its contents to GitHub.

## Successful initial backup

The initial full Restic backup completed successfully on 1 November 2026.

The run reported:

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

### Recorded result

| Metric | Result |
|---|---:|
| Files processed | 48,335 |
| Directories processed | 30,960 |
| Source data scanned | 197.721 GiB |
| Data added to repository | 136.272 GiB |
| Data stored | 135.725 GiB |
| Duration | 3 hours, 47 minutes, 13 seconds |
| Source-side VSS snapshot | Successfully created for `C:\` |
| Restic snapshot | `7ec7a616` |
| Restic exit code | `0` |

Restic reported that no parent snapshot existed, so it read all files for this initial backup. The `snapshot 7ec7a616 saved` message and exit code `0` confirm that Restic completed and saved the initial snapshot.

The figures for data added and data stored are different because Restic's repository storage uses compression and deduplication.

## Earlier interrupted attempt

An earlier full backup attempt failed while Restic was writing data to the external Seagate drive. Restic reported `A device which does not exist was specified`, followed by failed temporary-file operations and exit code 1.

Windows recorded a Disk 153 I/O retry at 18:21:20 on 1 November 2026, followed by Disk 51 warnings and NTFS Event 50 delayed-write failures at 18:25:42. NTFS later reported the volume healthy.

A subsequent `chkdsk D: /scan` found no filesystem problems and reported zero bad sectors. The device was later Online / Healthy. A later full Restic backup then completed successfully and saved snapshot `7ec7a616`.

The successful retry demonstrates that the repository accepted a complete initial backup. However, the precise cause of the earlier I/O interruption remains unresolved. It should not be attributed conclusively to the cable, USB port, enclosure or drive without further evidence.

See [Case Study: Investigating an Interrupted Immich Backup](case-studies/immich-backup-incident.md) for the diagnostic timeline and subsequent successful backup.

## Verification still required

The initial backup is complete, but the recovery process has not yet been fully tested.

The following checks remain:

1. Run `restic snapshots` and confirm that snapshot `7ec7a616` is listed.
2. Run a post-backup `restic check` against the repository.
3. Inspect the snapshot contents and confirm the expected directory roots and important files are present.
4. Restore a sample selection to a separate temporary directory and compare it with the source.
5. Perform a PostgreSQL restore test in an isolated test environment, not over the live database.
6. Record the repository password recovery procedure without recording the password itself.
7. Confirm the external drive and repository can be accessed from another Windows machine.
8. Establish a repeat schedule and retention policy after capacity and restore testing.

An earlier repository integrity check passed before the successful snapshot was created. That earlier result is not a substitute for a fresh check of the completed snapshot.

## Important recovery caveats

- The live PostgreSQL data directory should not be copied while PostgreSQL is running as if it were an ordinary static folder.
- The Immich original media directory and Immich-managed media directory are separate data sets in the observed layout; protect both unless a verified storage design proves otherwise.
- A single external drive connected to the same physical host is not an off-site backup. It protects against some host-drive failures but not theft, fire, ransomware affecting both disks, or accidental repository deletion.
- A successful backup run does not prove that every file can be restored or that Immich can be recovered successfully. Restore testing remains necessary.
- This document intentionally excludes exact passwords, `.env` values, private keys and certificate contents.
