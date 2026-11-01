# Evidence and Method

## Evidence standard

This documentation separates direct observations from interpretations.

- **Verified:** direct command output, container inspection, file listing or configuration content was supplied during the audit.
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
| Immich database backup | gzip stream decompression/integrity check |
| Restic | `restic version`, repository initialisation and dry-run backup output |
| Pi-hole | Mount inspection, `docker diff`, container status and upstream configuration |

## Important limitations

- All observations are point-in-time. The environment may have changed since the capture date.
- The inspection was primarily read-only. A separately recorded cleanup removed four Nextcloud-labelled Docker volumes after no matching Nextcloud containers remained.
- No complete restore drill has been completed.
- No external network scan, formal vulnerability scan or packet capture was performed.
- Host-port publication does not establish internet exposure.
- A `healthy` Docker status depends on the configured health check and is not a substitute for end-to-end testing.
- A running container does not prove that its data is persisted or its deployment can be recreated.
- Docker network membership does not prove that two applications communicate.
- Image tags do not uniquely identify immutable image contents; digests were not included in this initial inventory.
- The report does not contain secret values, token values, password hashes, `.env` contents, private keys or certificate contents.

## Reproducibility

When refreshing this audit, capture timestamped outputs and redact secrets before storing them. Prefer command output saved as text files in a private evidence folder. Store sanitized summaries in GitHub; keep raw outputs and private deployment files outside any public repository.

## Change record

| Date | Change |
|---|---|
| 2026-11-01 | Initial documentation baseline generated from discovery outputs supplied during the audit |
