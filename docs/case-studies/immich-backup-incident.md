# Case Study: Investigating an Interrupted Immich Backup

**Date:** 1 November 2026  
**Status:** Investigation documented; backup not yet completed  
**Environment:** Windows 11, Restic 0.19.1, external Seagate USB drive formatted as NTFS

## Objective

Create an encrypted external backup of the Immich photo/video library, application-managed media and selected deployment configuration. The target was an external Seagate Portable drive with approximately 1 TB capacity.

This case study records the first full-backup attempt and the diagnostic work that followed. It does not claim that the backup was successfully completed.

## Initial preparation

- Restic 0.19.1 was installed.
- A password file was created under the current Windows user's LocalAppData area and its access was restricted.
- A Restic repository was successfully initialised on the external drive. The repository ID prefix was `[PRIVATE_RESTIC_REPOSITORY_IDENTIFIER]`.
- A Volume Shadow Copy preflight/dry run completed successfully.
- The dry run scanned 48,335 files and 197.721 GiB. It estimated 153.785 GiB to add to the repository, with 153.182 GiB stored.

A dry run creates no completed Restic snapshot. It only demonstrated that the source scan and VSS preflight could complete at that time.

## Failure during the first full backup

The first actual backup attempt started and Restic reported that it had successfully created a snapshot for `C:\` using Volume Shadow Copy. During data writes to the external repository, Restic reported errors including:

- `A device which does not exist was specified` while syncing temporary data files under the repository's `data` directory;
- subsequent temporary-file opens failing because the expected path could not be found;
- `Fatal: unable to save snapshot`;
- process exit code `1`.

The phrase “successfully created snapshot for [c:\]” refers to the source-side VSS snapshot. It does not mean that Restic committed a completed backup snapshot to the repository.

## Windows event-log investigation

The Windows System log contained storage errors associated with the Seagate disk (Disk 1):

| Time on 1 November 2026 | Provider / event | Observed message | Interpretation |
|---|---|---|---|
| 18:21:20 | `disk`, Event 153 | An I/O operation at a logical block address on Disk 1 was retried | Windows had already encountered a storage I/O problem before the Restic failure |
| 18:25:42 | `disk`, Event 51 | Errors during disk paging operations | Further evidence of a disk I/O failure during the backup window |
| 18:25:42 | `Ntfs`, Event 50 | Delayed writes failed for repository data and NTFS metadata | Windows could not commit multiple writes to the external volume |
| 18:25:44 | `Microsoft-Windows-Ntfs`, Event 98 | Volume D: reported healthy | NTFS later reported the volume as healthy; this does not disprove the earlier I/O errors |

The repeated Disk 51 and NTFS 50 events aligned with the Restic write failures. The precise underlying cause remains undetermined. Potential causes include the USB cable/connector, host USB port/controller, enclosure/power behaviour or the drive itself.

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

These counters are not enough to diagnose the physical drive. In particular, the write-error counters were unavailable, and the USB bridge may expose only a subset of device telemetry. The 184 total read errors are recorded as a reported cumulative counter, not as proof of bad sectors or current unreadable data.

## Restic repository verification

After the failed backup, `restic snapshots --no-lock` opened the repository successfully and listed no snapshots. `restic check --no-lock` completed with exit code 0 and reported `no errors were found`.

Restic also reported 475 additional files in the repository, described by the tool as likely containing duplicate data. This is a non-critical repository observation, but it is not a completed backup. No `prune`, `unlock`, reinitialisation or destructive repository operation was performed.

### Verified state after investigation

| Check | Result |
|---|---|
| Repository can be opened | Passed |
| Repository integrity check | Passed; no errors reported |
| Completed Restic snapshots | Zero |
| Filesystem online scan | Passed |
| Windows disk status | Online / Healthy at latest check |
| New storage errors in the later observation window | None found in that query |
| Sustained write reliability | Not yet proven |
| Full Immich backup | Not completed |
| Restore test | Not performed |

## Lessons and next actions

1. A successful source-side VSS snapshot is not the same as a completed Restic repository snapshot.
2. A filesystem scan reporting no problems does not rule out intermittent device, USB or controller I/O faults.
3. A repository integrity check can pass even when the repository contains no usable backup snapshots.
4. The backup must not be described as operational until a full snapshot exists and a restore test succeeds.
5. Investigate the physical USB connection and observe the device under a controlled, non-critical write test before retrying the full media backup.
6. Do not prune or reinitialise the existing repository until its state and recovery requirements have been reviewed.

## Portfolio relevance

This investigation demonstrates an evidence-led troubleshooting process: capturing application errors, correlating them with operating-system event logs, checking filesystem consistency, inspecting device telemetry, validating repository structure and communicating the limits of what each check proves.

The key outcome is not a claim that the backup has been fixed. It is a clearly evidenced diagnosis with a defined set of verification steps still outstanding.
