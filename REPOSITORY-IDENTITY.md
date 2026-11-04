

# Repository Identity

## Repository details

| Property | Value |
|---|---|
| Repository owner | `Unplayable-debug` |
| Repository name | `self-hosted-infrastructure-lab` |
| Full repository identifier | `Unplayable-debug/self-hosted-infrastructure-lab` |
| Repository URL | https://github.com/Unplayable-debug/self-hosted-infrastructure-lab |
| Primary branch | `main` |
| Portfolio type | Documentation-led infrastructure portfolio |
| Documentation review date | 4 November 2026 |

## Purpose

This repository documents a self-hosted infrastructure environment and the practical work involved in operating, troubleshooting and improving it.

Its intended audience includes:

- Infrastructure and systems administration employers.
- IT support and technical operations employers.
- Network and platform engineering audiences.
- Other learners interested in self-hosted infrastructure.

The repository demonstrates practical understanding through architecture documentation, technical investigation, recovery testing and evidence-led reporting.

## Project scope

The portfolio covers:

- Windows 11 and Docker Desktop.
- Ubuntu on WSL 2.
- Containerised services.
- Local networking, DNS and reverse proxying.
- Remote-access technologies.
- Persistent data and backup design.
- Restic backup automation.
- PostgreSQL database recovery.
- Security observations and improvement planning.

## Repository content

The repository contains Markdown documentation, Mermaid diagrams and repository-level guidance.

It does not contain the actual deployment environment or the private configuration needed to recreate the live installation.

The absence of deployment manifests is intentional. This repository is designed to explain the infrastructure and the work performed, not to distribute a ready-to-run copy of a private home environment.

## Evidence policy

Claims should be supported by one of the following:

1. A directly observed configuration or system result.
2. A recorded test result.
3. Operator-provided evidence clearly identified as such.
4. A clearly labelled inference.

Untested functionality must not be represented as working.

Historical evidence must include its relevant date so that it is not mistaken for a live status report.

## Privacy and publication policy

The public portfolio should not contain:

- Credentials, tokens or passwords.
- Private keys or sensitive certificate material.
- Internal IP addresses or private DNS details unless specifically justified.
- Exact firewall rules or sensitive access-control configuration.
- Raw backup files, database dumps or private logs.
- Personal local filesystem paths.

Examples use generic paths and environment variables instead of personal Windows profile paths.

## Maintenance

The documentation should be updated when a material architecture change, security improvement, backup change or recovery test is completed.

The container inventory and current-state document are point-in-time records. They should not be interpreted as automatically updated monitoring information.

