# Container Inventory

**Snapshot date:** 1 November 2026  
**Scope:** 16 in-scope running containers; one separately deferred project is excluded.

Port notation is `host:container`. A dash means no host port was published in the observed `docker ps` output. Internal-only container ports may still be available to peers on Docker networks.

| Container | Image / image family | Published ports | Health / notes |
|---|---|---:|---|
| [CONTAINER ROLE] | [IMAGE DETAILS OMITTED] | [PUBLISHED PORT DETAILS OMITTED] | [HISTORICAL NOTES GENERALIZED] |
| [CONTAINER ROLE] | [IMAGE DETAILS OMITTED] | [PUBLISHED PORT DETAILS OMITTED] | [HISTORICAL NOTES GENERALIZED] |
| [CONTAINER ROLE] | [IMAGE DETAILS OMITTED] | [PUBLISHED PORT DETAILS OMITTED] | [HISTORICAL NOTES GENERALIZED] |
| [CONTAINER ROLE] | [IMAGE DETAILS OMITTED] | [PUBLISHED PORT DETAILS OMITTED] | [HISTORICAL NOTES GENERALIZED] |
| [CONTAINER ROLE] | [IMAGE DETAILS OMITTED] | [PUBLISHED PORT DETAILS OMITTED] | [HISTORICAL NOTES GENERALIZED] |
| [CONTAINER ROLE] | [IMAGE DETAILS OMITTED] | [PUBLISHED PORT DETAILS OMITTED] | [HISTORICAL NOTES GENERALIZED] |
| [CONTAINER ROLE] | [IMAGE DETAILS OMITTED] | [PUBLISHED PORT DETAILS OMITTED] | [HISTORICAL NOTES GENERALIZED] |
| [CONTAINER ROLE] | [IMAGE DETAILS OMITTED] | [PUBLISHED PORT DETAILS OMITTED] | [HISTORICAL NOTES GENERALIZED] |
| [CONTAINER ROLE] | [IMAGE DETAILS OMITTED] | [PUBLISHED PORT DETAILS OMITTED] | [HISTORICAL NOTES GENERALIZED] |
| [CONTAINER ROLE] | [IMAGE DETAILS OMITTED] | [PUBLISHED PORT DETAILS OMITTED] | [HISTORICAL NOTES GENERALIZED] |
| [CONTAINER ROLE] | [IMAGE DETAILS OMITTED] | [PUBLISHED PORT DETAILS OMITTED] | [HISTORICAL NOTES GENERALIZED] |
| [CONTAINER ROLE] | [IMAGE DETAILS OMITTED] | [PUBLISHED PORT DETAILS OMITTED] | [HISTORICAL NOTES GENERALIZED] |
| [CONTAINER ROLE] | [IMAGE DETAILS OMITTED] | [PUBLISHED PORT DETAILS OMITTED] | [HISTORICAL NOTES GENERALIZED] |
| [CONTAINER ROLE] | [IMAGE DETAILS OMITTED] | [PUBLISHED PORT DETAILS OMITTED] | [HISTORICAL NOTES GENERALIZED] |
| [CONTAINER ROLE] | [IMAGE DETAILS OMITTED] | [PUBLISHED PORT DETAILS OMITTED] | [HISTORICAL NOTES GENERALIZED] |
| [CONTAINER ROLE] | [IMAGE DETAILS OMITTED] | [PUBLISHED PORT DETAILS OMITTED] | [HISTORICAL NOTES GENERALIZED] |

## Persistence and mounts

[Historical per-workload mount and persistence details omitted.]
## Restart and health observations

- Immich server showed restart count 2 in an earlier inspection. During the latest snapshot, Immich was healthy. The user confirmed that the temporary outage and Caddy 502 entries were caused by deliberate Immich image maintenance.
- Pi-hole showed restart count 21 in an earlier inspection, while remaining healthy with zero current health-check failure streak. The reason for historical restarts is not established.
- Health status is not equally configured across containers. A container without a health check can be running while its application is unhealthy.

## Image and tag policy observations

Several services use moving tags such as `latest`, `stable`, `release` or `main`. These are convenient for updates but make repeatable rollbacks harder unless the exact deployed digest is recorded. Capture image IDs/digests during each release and consider pinning tags or digests where reproducibility matters.

## Inventory limitations

This table is based on the observed running-container snapshot and earlier inspect outputs. It does not include every image, stopped container, build-cache item, anonymous volume or historical project. It is not a licence to delete unused resources.
