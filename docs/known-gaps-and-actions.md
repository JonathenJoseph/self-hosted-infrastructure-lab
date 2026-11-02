# Known Gaps and Actions

**Last updated:** 2 November 2026

This document tracks technical gaps, recovery risks and outstanding improvements identified during the infrastructure audit.

It distinguishes completed work from tasks that still require testing. Listing an action does not constitute permission to change the live environment.

## 1. Priority 0: backup and recovery

| Item | Evidence | Required action | Status |
|---|---|---|---|
| Initial Immich backup | Snapshot `7ec7a616` saved successfully | Preserve the initial recovery point | Complete |
| Subsequent backup | Snapshot `e8d1bd6a` saved successfully | Continue daily backups | Initial second snapshot complete |
| Repository structural check | Two snapshots checked; no errors reported | Perform periodic checks | Initial check passed |
| Additional repository files | Restic reported 475 additional files | Investigate and clean up only after further verification | Open |
| File recovery | Restored dump matched original SHA-256 | Retain test evidence | Complete |
| PostgreSQL recovery | 72 base tables and eight extensions verified | Document recovery results | Complete |
| Full application recovery | Database import succeeded, but Immich application not started against restored database | Perform an isolated Immich recovery drill | Open |
| Full repository data check | Normal `restic check` performed | Run `restic check --read-data` when appropriate | Open |
| External drive reliability | Earlier Disk and NTFS errors; subsequent full backup succeeded | Monitor future backup runs and investigate recurrence | Monitoring |
| Password recovery | Password file exists with restricted ACLs | Confirm secure off-device recovery copy | Open |

## 2. Priority 1: daily backup automation

| Item | Evidence | Required action | Status |
|---|---|---|---|
| Task Scheduler | Daily task configured for 05:00 | Keep schedule active | Configured |
| Latest PowerShell script | Includes VSS, logging, freshness checks, snapshot verification and retention | Validate updated version | Implemented; operational test pending |
| Database freshness | Script requires a current-day database dump | Confirm Immich produces the daily dump before 05:00 | Open |
| Dry-run after midnight | Correctly refused to back up yesterday's dump | Retain fail-safe behaviour | Verified |
| First scheduled run with latest script | Not yet confirmed | Check task result and Restic snapshots after scheduled execution | Open |
| Automatic retention | Included in updated script | Verify after a successful scheduled run | Pending |
| Concurrent execution | Task configured to prevent overlapping runs | Preserve setting | Configured |
| Failure recovery | Task Scheduler restart and missed-run options configured | Confirm actual behaviour during a controlled failure test | Open |

## 3. Priority 2: network and deployment reproducibility

| Item | Evidence | Required action | Status |
|---|---|---|---|
| Caddy route coverage | Nine site snippets observed | Verify application reachability and upstream ownership | Partially documented |
| Kali route | Upstream port 8088 unresolved | Identify the intended service or remove the obsolete route | Open |
| Open WebUI route label | `ollama` snippet points to the Open WebUI host port | Confirm intended route naming | Open |
| Pi-hole deployment | Active container has no mounts | Export configuration and plan persistent storage | Open |
| Pi-hole Compose definition | Referenced file absent | Locate or reconstruct deployment definition after preserving current state | Open |
| Pi-hole and Unbound relationship | Pi-hole uses public upstreams, Unbound runs separately | Confirm intended DNS architecture | Open |
| Standalone containers | Several containers lack Compose project labels | Identify reproducible deployment definitions | Open |

## 4. Priority 3: security and operations

| Item | Evidence | Required action | Status |
|---|---|---|---|
| Homepage Docker socket | Read-only Docker socket mount observed | Evaluate a restricted socket proxy or alternative integration | Open |
| Docker Desktop firewall rule | Broad backend firewall exception observed | Verify requirements and narrow safely if appropriate | Open |
| HTTP/HTTPS firewall rules | Duplicate broad rules identified | Review for redundancy and remove only after testing | Open |
| Nextcloud firewall rules | Old rules remain after Nextcloud removal | Confirm they are unused before removal | Open |
| AnyDesk firewall rules | Obsolete executable paths and unused process observations | Confirm they are no longer required | Open |
| Container privileges | Some services observed running as root | Evaluate non-root operation and volume permissions | Open |
| Container health checks | Several containers lack health checks | Add application-appropriate health checks where supported | Open |
| Image pinning | Moving image tags observed | Record immutable digests and establish rollback procedures | Open |

## 5. Completed work

The following activities have been completed:

- Nextcloud containers were absent during the audit.
- Four Nextcloud-named Docker volumes were removed at the user's explicit request.
- Immich was returned to service after intentional image maintenance.
- Caddy configuration validation passed.
- An initial encrypted Restic backup completed successfully.
- A second Restic snapshot was created in the same repository.
- Repository structural checking found no errors.
- A database dump was restored from Restic.
- Original and restored SHA-256 hashes matched.
- The recovered database was imported into isolated PostgreSQL.
- 72 base tables and eight extensions were confirmed.
- Temporary PostgreSQL test resources were removed.
- A manual Restic retention policy was applied without deleting either existing snapshot.
- Twingate access to any service on the home network was confirmed by the operator.
- Windows Task Scheduler was configured for daily execution.

## 6. Change management principles

Before modifying a live service:

1. Confirm the current configuration and service owner.
2. Identify persistent data and dependencies.
3. Confirm a working backup exists.
4. Document the proposed change and expected impact.
5. Define a rollback method.
6. Validate the result after the change.

Do not delete an unused image, volume or configuration directory solely because it is not currently attached to a running container.
