

# Security Baseline

## 1. Purpose

This document records the security principles and selected observations relevant to the self-hosted environment.

It is a practical baseline, not a certification, penetration-test report or claim that the environment is free from vulnerabilities.

## 2. Security objectives

The main objectives are:

- Protect credentials and private configuration.
- Limit unnecessary service exposure.
- Separate application and supporting services where practical.
- Protect persistent data.
- Maintain encrypted backups.
- Test restoration rather than relying only on successful backup creation.
- Avoid publishing private infrastructure details.
- Record unresolved weaknesses honestly.

## 3. Secrets and credentials

Credentials and private configuration should remain outside the public Git repository.

Examples include:

- Environment variables and passwords.
- API tokens and connector credentials.
- Private keys.
- Database credentials.
- Backup repository passwords.
- Private certificate material.

The Restic repository password is not published in this portfolio.

The `.gitignore` file provides common protection against accidentally adding environment files, keys, database dumps and backup archives. It is not a substitute for reviewing staged files before committing.

## 4. Container security

Containers should run with only the access they require.

Security reviews should consider:

- Container privileges.
- Host filesystem mounts.
- Docker socket access.
- Published ports.
- Network membership.
- Persistent storage.
- Image provenance and updates.
- Service authentication.

### Docker socket access

The reviewed dashboard configuration used read-only Docker socket access.

Read-only access does not automatically make Docker API access low risk. The API can expose substantial information and capabilities.

The long-term objective should be to minimise Docker API access and use a narrower integration where practical.

### Persistent storage

Application state should use deliberate persistent storage or a documented backup mechanism.

The reviewed Pi-hole configuration did not have dedicated persistent mounts for its application state. This was recorded as a persistence risk requiring improvement and revalidation.

### Network exposure

Some services were observed publishing host ports on all interfaces.

This does not prove public internet accessibility. Exposure must be assessed using the host firewall, gateway configuration, network boundaries and service access controls together.

## 5. Network security

The network architecture separates:

- Local DNS and LAN routing.
- Application-level reverse proxying.
- Container communication.
- Remote-access connectivity.

The public documentation does not publish detailed firewall rules, private addressing or remote-access resource configuration.

Remote access is provided through independently configured Twingate and UniFi Teleport options.

The presence of a remote-access tool does not, by itself, prove that every service is protected correctly. Access policies and service authentication still need appropriate review.

## 6. Backup security

The Restic repository uses encryption.

The backup password is stored separately from the public documentation, and the local password file is protected with Windows access-control permissions.

Security and recovery considerations include:

- Protecting the repository password.
- Maintaining a separate recoverable copy of the password.
- Protecting the external backup medium.
- Verifying snapshots after creation.
- Testing actual restores.
- Avoiding unsafe retention or pruning operations.
- Keeping database dumps out of the public repository.

The recorded full data-pack read completed successfully, and a database dump was restored and imported into an isolated PostgreSQL test environment.

Complete Immich application recovery remains untested.

## 7. Repository security

Before publication, the repository should be checked for:

- Secrets in current files.
- Secrets in historical commits.
- Personal information in file contents and paths.
- Private operational identifiers.
- Sensitive diagrams or screenshots.
- Accidentally committed logs or backup data.

A dedicated Gitleaks full-history scan is still required to strengthen the secret-scanning evidence.

The commit author email is intentionally accepted by the repository owner as public Git metadata.

Historical local filesystem paths are a separate privacy consideration. Cleaning current files does not remove older versions from Git history.

## 8. Security claims and limitations

This portfolio does not claim:

- Successful penetration testing.
- Complete vulnerability elimination.
- Enterprise security certification.
- Guaranteed internet isolation.
- Complete application disaster recovery.
- That every configured network route has been tested.
- That every historical security observation still applies.

Security statements are limited to the evidence available and the date of the observation.

## 9. Improvement priorities

| Priority | Improvement |
|---|---|
| High | Minimise Docker socket access |
| High | Ensure application state is persistently stored or backed up |
| High | Revalidate host-published services and firewall boundaries |
| High | Verify backup password recovery arrangements |
| Medium | Review container privileges and network separation |
| Medium | Validate important reverse-proxy upstreams |
| Medium | Review image update and version-pinning strategy |
| Medium | Complete dedicated secret scanning before publication |

