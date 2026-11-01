# Backup and Recovery

**Snapshot date:** 1 November 2026  
**Status:** initial full Restic backup attempt failed during writes to the external drive; repository integrity check passed, but zero completed snapshots and no restore test exist.

## Immich data locations and approximate size

| Data set | Approximate size | Observed content | Backup handling |
|---|---:|---|---|
| Original photos and videos | 129.83 GiB | About 19,576 files | Include in file-level backup |
| Immich media directory | 67.89 GiB | About 28,755 files, including application-managed media and PostgreSQL dumps | Include in backup; understand subdirectories before selective exclusions |
| PostgreSQL live data directory | 0.55 GiB | Live database files | Do not copy as an ordinary live directory; use database dumps or a consistent database-aware backup |
| Separate storage directory | Empty at inspection | No files found | Re-check before assuming it remains unused |

These measurements are point-in-time estimates, not capacity guarantees.

## PostgreSQL dump verification

A manual Immich database dump was created on 1 November 2026. The gzip file was approximately 97 MB. It was decompressed successfully to approximately 238 MiB, and gzip integrity passed.

This verifies archive-stream integrity only. It does **not** prove that the dump can be restored by PostgreSQL or that the restored Immich application starts correctly. A restore test remains outstanding.

Fourteen daily compressed database dump files were observed in the Immich backups directory during the initial inspection. Retention and automatic dump generation should be confirmed.

## Restic preparation

| Item | Observed state |
|---|---|
| Restic | Version 0.19.1 installed |
| External target | Seagate USB drive, mounted as `D:` at inspection |
| Repository | `[LOCAL_BACKUP_REPOSITORY_PATH]` |
| Repository initialisation | Successful; repository ID prefix recorded as `[PRIVATE_RESTIC_REPOSITORY_IDENTIFIER]` |
| Password file | Created under the current Windows user's LocalAppData Restic directory and ACL-restricted |
| Dry run | Successful with VSS snapshot support |
| Actual full snapshot | **Failed on first attempt**; Restic exit code 1 after Windows storage write errors |
| Repository check after failure | Passed with no integrity errors; Restic reported 475 additional files; zero snapshots |
| Restore test | **Not performed** |

The successful dry run reported 48,335 files and 197.721 GiB scanned. It estimated 153.785 GiB to add to the repository (153.182 GiB stored). A dry run does not store a backup snapshot.

## First full-backup attempt and investigation

The first actual backup attempt failed while Restic was writing data to the external Seagate drive. Restic reported `A device which does not exist was specified`, followed by failed temporary-file operations and exit code 1. The successful message about creating a snapshot for `C:\` referred to the source-side Volume Shadow Copy, not a completed Restic repository snapshot.

Windows recorded a Disk 153 I/O retry at 18:21:20 on 1 November 2026, followed by repeated Disk 51 warnings and NTFS Event 50 delayed-write failures at 18:25:42. NTFS later reported the volume healthy. A subsequent `chkdsk D: /scan` found no filesystem problems and reported zero bad sectors. The device was later Online / Healthy, and a later short event-log query found no new storage warnings in that observation window.

After the failure, `restic snapshots --no-lock` listed zero snapshots. `restic check --no-lock` returned exit code 0 and reported no integrity errors, while also reporting 475 additional files in the repository. No prune, unlock or reinitialisation was performed.

The exact cause remains unresolved. An intermittent USB connection, port/controller, enclosure/power or drive issue remains possible. The external repository must not be counted as a backup until a full snapshot and restore test succeed. See [Case Study: Investigating an Interrupted Immich Backup](case-studies/immich-backup-incident.md) for the diagnostic timeline and detailed evidence.

The backup password must be recoverable independently of the Windows installation. Keep a recovery copy in a password manager or other secure off-device location. Never commit the password file or its contents to GitHub.

## Recommended backup contents

The prepared initial backup selection included:

- Original photos and videos
- [Historical per-service storage and persistence details omitted.]
- Immich `.env` file
- Immich Compose file
- Caddyfile and Caddy site configuration
- Caddy root certificate file

The Restic repository is encrypted, but source configuration may contain credentials, private paths or internal hostnames. Keep it encrypted in the backup and exclude it from public GitHub commits.

## Recovery requirements

Before declaring the backup operational, complete all of the following:

1. Confirm that an actual Restic snapshot exists.
2. Run `restic check` against the repository.
3. Inspect the snapshot contents and confirm expected directory roots and file counts.
4. Restore a sample selection to a separate temporary directory and compare it with the source.
5. Perform a PostgreSQL restore test in an isolated test environment, not over the live database.
6. Record the repository password recovery procedure without recording the password itself.
7. Confirm the external drive can be identified and mounted on another Windows machine.
8. Establish a schedule and retention policy only after capacity and restore testing.

## Important recovery caveats

- The live PostgreSQL data directory should not be copied while PostgreSQL is running as if it were an ordinary static folder.
- The Immich original media directory and Immich-managed media directory are separate data sets in the observed layout; protect both unless a verified storage design proves otherwise.
- A single external drive connected to the same physical host is not an off-site backup. It protects against some host-drive failures but not theft, fire, ransomware affecting both disks, or accidental repository deletion.
- This document intentionally excludes exact passwords, `.env` values, private keys and certificate contents.
