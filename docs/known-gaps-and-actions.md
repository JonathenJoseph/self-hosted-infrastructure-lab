

# Known Gaps and Actions

## 1. Purpose

This document records known limitations, unresolved observations and planned improvement areas.

An item should only be marked complete when there is evidence that the relevant work has been performed.

## 2. Backup and recovery

| ID | Gap | Current status | Next action |
|---|---|---|---|
| BK-01 | Complete Immich application recovery | Not tested | Restore database and media into an isolated environment, start the application and verify media access |
| BK-02 | 475 additional Restic files | Unresolved observation | Establish their origin and status before considering repository maintenance |
| BK-03 | Automatic pruning | Disabled | Keep disabled until repository state and recovery confidence justify enabling it |
| BK-04 | Independent password recovery | Not established by available evidence | Confirm that a secure, separate recovery copy of the repository password exists and can be accessed |
| BK-05 | Future scheduled runs | Require ongoing verification | Review subsequent Task Scheduler results and snapshot records |

The successful full data-pack read does not resolve BK-02.

No pruning should be performed solely because the earlier repository check reported additional files.

## 3. Storage incident

| ID | Gap | Current status | Next action |
|---|---|---|---|
| ST-01 | Cause of earlier storage write errors | Unresolved | Continue monitoring storage behaviour and record any recurrence |
| ST-02 | Independent hardware diagnosis | Not established | Avoid claiming a confirmed disk fault without supporting evidence |

A later filesystem scan reported no filesystem problems and no bad sectors. Subsequent backups succeeded, but the original cause was not established.

## 4. Container and application security

| ID | Area | Observation | Improvement |
|---|---|---|---|
| SEC-01 | Docker API access | Read-only Docker socket access still exposes a broad API surface | Review whether the integration can be replaced with a narrower access method |
| SEC-02 | Pi-hole persistence | Application state was not backed by dedicated persistent mounts in the reviewed configuration | Configure and test persistent storage or a reliable state backup |
| SEC-03 | Published ports | Some services published host ports on all interfaces in the reviewed configuration | Review host firewall and network boundaries; verify which services actually need host publication |
| SEC-04 | Container isolation | Several services use shared or general-purpose Docker networks | Review whether stronger service separation is appropriate |
| SEC-05 | Reverse-proxy routes | Not every configured upstream was independently tested | Validate application connectivity and document the result |

These observations are based on a previous configuration review. They must be revalidated before being described as the current state.

## 5. Network and remote access

| ID | Area | Status | Next action |
|---|---|---|---|
| NET-01 | Twingate remote access | Configured for access to services across the home network | Revalidate access when making material network changes |
| NET-02 | UniFi Teleport | Separate remote VPN option | Retain independent documentation and avoid implying that it depends on Twingate |
| NET-03 | DNS architecture | UniFi local DNS/routing and Caddy application routing are separate | Preserve this distinction in future documentation |
| NET-04 | Upstream connectivity | Configuration is not proof of reachability | Test important routes from the relevant network locations |

## 6. Documentation improvements

| ID | Improvement | Status |
|---|---|---|
| DOC-01 | Keep the container inventory clearly marked as a point-in-time snapshot | Required during documentation updates |
| DOC-02 | Keep internal paths generic and free from personal profile names | Applied in the public documentation revision |
| DOC-03 | Keep detailed port mappings and firewall rules out of the public portfolio | Applied in the public documentation revision |
| DOC-04 | Keep backup dates and evidence provenance explicit | Required during documentation updates |
| DOC-05 | Validate Mermaid syntax | Outstanding |
| DOC-06 | Recheck internal Markdown links after documentation changes | Outstanding |

## 7. Publication verification

The following checks should be completed before the repository is made public:

| Check | Status |
|---|---|
| Current documentation privacy review | Required |
| Historical Windows profile reference decision | Outstanding |
| Dedicated Gitleaks full-history scan | Outstanding |
| Mermaid parser validation | Outstanding |
| Internal Markdown link validation after edits | Outstanding |
| GitHub security feature review | Outstanding |
| Repository collaborator review | Outstanding |
| Security reporting route | Enable private vulnerability reporting where available |
| Final public-facing review | Outstanding |

The repository owner has accepted the personal author email being visible in Git commit metadata. No history rewrite is required solely to remove that email.

Historical Windows profile references are a separate privacy decision. Replacing the current documentation does not remove older versions from Git history.

## 8. Items deliberately not marked as complete

The following must remain open until further evidence is available:

- Full Immich application recovery.
- Explanation of the additional Restic files.
- The cause of the original storage failure.
- Automatic pruning.
- Confirmation of a tested password recovery copy.
- Revalidation of the container security observations.

