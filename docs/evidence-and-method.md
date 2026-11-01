# Evidence and Method

## Evidence standard

This documentation separates direct observations from interpretations.

- **Verified:** direct command output, container inspection, file listing or configuration content was supplied during the audit.
- **User-confirmed:** explicitly confirmed by the operator.
- **Inferred:** a conclusion follows from multiple observations but has not been tested directly.
- **Open:** further evidence or user intent is required.
- **Deferred:** intentionally excluded from the current scope.

## Commands and evidence classes used

| Area | Evidence gathered |
|---|---|
| Host | Windows build/system information, CPU and memory inventory, storage capacity |
| WSL | `wsl --version`, distribution state and Docker CLI/server identity |
| Docker | `docker version`, `docker ps -a`, `docker inspect`, `docker stats`, `docker compose ls`, `docker network ls/inspect`, `docker volume ls/inspect`, `docker system df -v` |
| Compose | `docker compose config` validation and service discovery for available project files |
| Caddy | Main Caddyfile, site snippets, network membership, `caddy validate`, recent logs |
| Firewall | `Get-NetConnectionProfile`, `Get-NetFirewallProfile`, `Get-NetFirewallRule` with port/address/application filters |
| Storage | Host directory sizes and mount inspection |
| Immich database backup | gzip stream decompression and integrity check |
| Restic preparation | `restic version`, repository initialisation and dry-run backup output |
| Restic backup | Full backup output showing VSS snapshot creation, files and directories processed, data stored, saved snapshot ID and exit code |
| Windows storage investigation | Disk and NTFS System event logs, `chkdsk D: /scan`, device health observations |
| Pi-hole | Mount inspection, `docker diff`, container status and upstream configuration |
| Remote access | Operator confirmation that Twingate provides access to any service on the home network |

## Backup evidence recorded

The successful initial Restic backup output reported:

- Repository opened successfully.
- A source-side VSS snapshot for `C:\` was created successfully.
- 48,335 files and 30,960 directories processed.
- 197.721 GiB scanned.
- 136.272 GiB added to the repository, with 135.725 GiB stored.
- Processing completed in 3 hours, 47 minutes and 13 seconds.
- Snapshot `7ec7a616` saved.
- Backup exit code `0`.

This supports the claim that the initial backup run completed and saved a Restic snapshot.

It does not establish that a restore has been tested. A post-backup repository check, snapshot inspection and sample restore remain outstanding.

## Remote-access evidence

The operator confirmed that Twingate allows remote access to any service on the home network. This is recorded as user-confirmed configuration information.

The Twingate Connector was also observed in the Docker inventory using host networking. Credential values are intentionally excluded from this portfolio.

UniFi Teleport is documented as a separate remote VPN option through WiFiman.

## Important limitations

- All observations are point-in-time. The environment may have changed since the capture date.
- The inspection was primarily read-only. A separately recorded cleanup removed four Nextcloud-labelled Docker volumes after no matching Nextcloud containers remained.
- No complete application restore drill has been completed.
- No external network scan, formal vulnerability scan or packet capture was performed.
- Host-port publication does not establish internet exposure.
- A `healthy` Docker status depends on the configured health check and is not a substitute for end-to-end testing.
- A running container does not prove that its data is persisted or its deployment can be recreated.
- Docker network membership does not prove that two applications communicate.
- Image tags do not uniquely identify immutable image contents; digests were not included in this initial inventory.
- The successful Restic backup does not by itself prove that the external storage will remain reliable across future runs.
- The report does not contain secret values, token values, password hashes, `.env` contents, private keys or certificate contents.

## Reproducibility

When refreshing this audit, capture timestamped outputs and redact secrets before storing them. Prefer command output saved as text files in a private evidence folder. Store sanitised summaries in GitHub; keep raw outputs and private deployment files outside any public repository.

## Change record

| Date | Change |
|---|---|
| 2026-11-01 | Initial documentation baseline generated from discovery outputs supplied during the audit |
| 2026-11-01 | Updated backup records after successful initial Restic snapshot `7ec7a616` was saved |
| 2026-11-01 | Updated remote-access documentation to record operator-confirmed Twingate access to any home-network service |
