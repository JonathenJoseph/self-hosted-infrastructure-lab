# Known Gaps and Actions

**Last reviewed:** 1 November 2026

This file tracks evidence-backed documentation and reliability tasks. It is not authorisation to perform changes.

## Priority 0: protect critical state

| Item | Evidence | Required outcome | Status |
|---|---|---|---|
| Immich Restic backup | Initial full backup completed; snapshot `7ec7a616` saved; 48,335 files and 197.721 GiB processed | Run a post-backup integrity check, inspect snapshot contents and test restoration | Snapshot completed; verification open |
| Immich PostgreSQL dump | Gzip integrity passed; database restore not tested | Restore dump in an isolated PostgreSQL instance | Open |
| Immich recovery procedure | Initial Restic snapshot exists; no complete recovery drill performed | Restore sample files and test the application recovery sequence | Open |
| External storage reliability | First full write attempt produced Disk 153, Disk 51 and NTFS 50 errors; a subsequent full backup succeeded | Monitor subsequent backup runs and investigate any recurrence | Initial retry successful; cause unresolved |
| Pi-hole state | No mounts; important files appear in writable container layer | Export Teleporter backup and verify recovery path | Open |
| Pi-hole deployment definition | Active Pi-hole has no Compose labels; referenced Compose file absent | Reconstruct or locate definition only after state export | Open |

## Priority 1: reproducible deployment documentation

| Item | Evidence | Required outcome | Status |
|---|---|---|---|
| Standalone containers | Six in-scope containers lacked Compose project labels | Locate source deployment definitions or create controlled Compose definitions | Open |
| Caddy routing | Nine snippets observed; Kali upstream unresolved | Verify each route, intended hostname and service owner | Partially documented |
| DNS architecture | Pi-hole configured with public upstreams; Unbound running separately | Confirm whether Unbound is intended to be Pi-hole's upstream | Open |
| Jellyfin mount layout | Parent media path and cache/config subfolders mounted | Confirm that media root does not unintentionally include app state | Open |
| Host port exposure | Multiple services published on all host interfaces | Document intended access scope for each service | Open |

## Priority 2: security and operations

| Item | Evidence | Required outcome | Status |
|---|---|---|---|
| Homepage Docker socket | Read-only Docker socket mount | Evaluate restricted socket proxy or remove integration if unnecessary | Open |
| Docker Desktop firewall rule | Backend inbound TCP/UDP Any on Private/Public | Verify dependencies and narrow safely if possible | Open |
| HTTP/HTTPS firewall rules | Duplicate broad inbound rules | Remove redundant rules after exporting a baseline | Open |
| Nextcloud firewall rules | Old Nextcloud allow rules remain after volume/container removal | Remove if confirmed unused | Open |
| AnyDesk firewall rules | Executable paths absent and process not running | Remove orphaned entries if AnyDesk is no longer needed | Open |
| Container privilege | Homepage and Open WebUI observed as root | Check non-root support and volume permissions | Open |
| Health checks | Several containers have no health check | Add application-appropriate checks where supported | Open |
| Image pinning | Moving tags in use | Record image digests and establish update/rollback process | Open |

## Completed or explained

- Nextcloud containers were absent; four Nextcloud-named Docker volumes were removed at the user's explicit request. The corresponding volume-name filter returned no remaining matches.
- Immich proxy 502 errors recorded during the Caddy log review were explained by the user as expected errors during intentional Immich maintenance. Immich was subsequently reported back online.
- Caddy configuration validation passed.
- The initial Immich Restic backup completed successfully and saved snapshot `7ec7a616`.
- The successful Restic run processed 48,335 files and 197.721 GiB, storing 135.725 GiB in the repository.
- The earlier failed Restic write attempt was investigated using Windows Disk and NTFS event logs. A subsequent full backup succeeded, although the original I/O interruption's cause remains unresolved.
- Twingate remote access to any service on the home network was confirmed by the operator.
