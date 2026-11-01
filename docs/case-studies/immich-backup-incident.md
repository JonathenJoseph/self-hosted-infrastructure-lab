# Case Study: Investigating an Interrupted Immich Backup

**Date:** 1 November 2026  
**Status:** Initial backup subsequently completed successfully; restore verification outstanding  
**Environment:** Windows 11, Restic 0.19.1, external Seagate USB drive formatted as NTFS

## Objective

Create an encrypted external backup of the Immich photo/video library, application-managed media and selected deployment configuration. The target was an external Seagate Portable drive with approximately 1 TB capacity.

This case study records the initial backup failure, the investigation and the subsequent successful full backup.

## Initial preparation

- Restic 0.19.1 was installed.
- A password file was created under the current Windows user's LocalAppData area and its access was restricted.
- A Restic repository was successfully initialised on the external drive. The repository ID prefix was `[PRIVATE_RESTIC_REPOSITORY_IDENTIFIER]`.
- A Volume Shadow Copy preflight/dry run completed successfully.
- The dry run scanned 48,335 files and 197.721 GiB.

A dry run creates no completed Restic snapshot. It only demonstrated that the source scan and VSS preflight could complete at that time.

## Failure during the first full backup

The first actual backup attempt started and Restic reported that it had successfully created a snapshot for `C:\` using Volume Shadow Copy.

During data writes to the external repository, Restic reported errors including:

- `A device which does not exist was specified` while syncing temporary data files under the repository's `data` directory;
- subsequent temporary-file opens failing because the expected path could not be found;
- `Fatal: unable to save snapshot`;
- process exit code `1`.

The phrase “successfully created snapshot for [c:\]” referred to the source-side VSS snapshot. It did not mean that Restic had committed a completed backup snapshot to the repository.

## Windows event-log investigation

The Windows System log contained storage errors associated with the Seagate disk:

| Time on 1 November 2026 | Provider / event | Observed message | Interpretation |
|---|---|---|---|
| 18:21:20 | `disk`, Event 153 | An I/O operation at a logical block address on Disk 1 was retried | Windows had encountered a storage I/O problem before the Restic failure |
| 18:25:42 | `disk`, Event 51 | Errors during disk paging operations | Further evidence of disk I/O failure during the backup window |
| 18:25:42 | `Ntfs`, Event 50 | Delayed writes failed for repository data and NTFS metadata | Windows could not commit multiple writes to the external volume |
| 18:25:44 | `Microsoft-Windows-Ntfs`, Event 98 | Volume D: reported healthy | NTFS later reported the volume as healthy; this does not disprove the earlier I/O errors |

The repeated Disk 51 and NTFS 50 events aligned with the Restic write failures. The precise underlying cause was not established. Potential causes included the USB cable or connector, host USB port/controller, enclosure or power behaviour, or the drive itself.

## Filesystem and device checks

A subsequent `chkdsk D: /scan` completed and reported:

- Windows scanned the filesystem and found no problems.
- Zero bad file records.
- Zero bad sectors reported by this filesystem scan.
- No further action required by CHKDSK.

Windows later reported the Seagate disk as Online and Healthy, and the D: volume as NTFS / Healthy / OK. A later ten-minute event-log query showed a healthy-volume notification but no new Disk, NTFS, UASPStor or Kernel-PnP warnings in that observation window.

Storage reliability counters exposed by Windows reported:

| Counter | Reported value |
|---|---:|
| Temperature | 36°C |
| Power-on hours | 657 |
| Total read errors | 184 |
| Uncorrected read errors | 0 |
| Write-error counters | Not reported |

These counters are not enough to diagnose the physical drive. In particular, write-error counters were unavailable, and the USB bridge may expose only a subset of device telemetry. The 184 total read errors are recorded as a reported cumulative counter, not as proof of bad sectors or current unreadable data.

## Successful subsequent backup

A later full backup run completed successfully on 1 November 2026.

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

### Successful run results

| Check | Result |
|---|---|
| Repository opened | Passed |
| Source-side VSS snapshot for `C:\` | Successfully created |
| Files processed | 48,335 |
| Directories processed | 30,960 |
| Data scanned | 197.721 GiB |
| Data added to repository | 136.272 GiB |
| Data stored | 135.725 GiB |
| Duration | 3 hours, 47 minutes, 13 seconds |
| Restic snapshot saved | `7ec7a616` |
| Restic exit code | `0` |

This confirms that the initial Restic backup completed and a snapshot was saved. The earlier I/O failure remains part of the incident history, but it did not prevent the subsequent backup from completing.

The successful retry does not establish the precise cause of the original storage interruption or guarantee future USB write reliability.

## Current verification state

| Check | Result |
|---|---|
| Repository initialisation | Passed |
| Successful initial Restic snapshot | Passed |
| Snapshot identifier | `7ec7a616` |
| Restic exit code | `0` |
| Earlier repository integrity check | Passed before successful snapshot was created |
| Post-backup repository integrity check | Outstanding |
| Snapshot contents inspection | Outstanding |
| Sample file restore | Outstanding |
| PostgreSQL restore test | Outstanding |
| Full Immich recovery drill | Outstanding |
| Cause of original USB I/O interruption | Unresolved |

## Lessons and next actions

1. A successful source-side VSS snapshot is not the same as a completed Restic repository snapshot.
2. A filesystem scan reporting no problems does not rule out intermittent device, USB or controller I/O faults.
3. A repository integrity check can pass even when the repository contains no usable backup snapshots.
4. A successful Restic snapshot confirms that the backup run completed, but not that a full application recovery has been tested.
5. Run a fresh repository integrity check and inspect the saved snapshot.
6. Restore sample files to a separate location and compare them with the source.
7. Perform a PostgreSQL restore test in an isolated environment.
8. Monitor the external storage during subsequent backup runs because the original I/O interruption remains unexplained.

## Portfolio relevance

This investigation demonstrates an evidence-led troubleshooting process: capturing application errors, correlating them with operating-system event logs, checking filesystem consistency, inspecting device telemetry, validating repository structure, completing a successful backup retry and communicating the limits of what each check proves.

The final outcome is a successful initial backup, accompanied by a clear record of the earlier failure and the recovery tests still required.
