

# Evidence and Method

## 1. Purpose

This document explains how technical claims in the portfolio are classified and how test results should be interpreted.

The objective is to avoid presenting assumptions as facts or describing an untested feature as working.

## 2. Evidence labels

The following labels are used throughout the portfolio.

| Label | Meaning |
|---|---|
| **OBSERVED** | Directly seen in a configuration, command output, application interface or system record during an environment review. |
| **OPERATOR-REPORTED** | Supplied by the system operator as a result or observation. It has not necessarily been independently repeated during documentation preparation. |
| **TESTED** | A test was carried out and a result was recorded. The test conditions and limitations should be stated. |
| **INFERRED** | A conclusion drawn from available evidence but not directly confirmed. |
| **NOT TESTED** | The relevant functionality has not been demonstrated. |

A single activity may have more than one evidence classification. For example, an operator may report a successful test and provide its output, while the documentation author has not re-executed it.

## 3. Evidence sources

The portfolio draws on:

- Windows system information and event records.
- Docker and container configuration observations.
- Network and reverse-proxy configuration.
- Restic command output and snapshot records.
- Windows Task Scheduler information.
- Database restore results.
- SHA-256 file hash comparisons.
- PostgreSQL import and query results.
- Operator-provided descriptions of the environment.

Private logs, credentials, configuration secrets and raw backup data are not included in the public repository.

## 4. Backup evidence

The backup results recorded for 1–2 November 2026 include:

- An initial full Restic snapshot.
- Subsequent daily snapshots.
- A successful scheduled backup on 2 November.
- Successful execution of the configured retention operation.
- Three snapshots remaining after the recorded retention run.
- A full data-pack read of 9,221 out of 9,221 packs with no reported errors.

These are historical operator-confirmed results. They should not be interpreted as live repository status.

## 5. Database recovery evidence

The database recovery test included:

1. Restoring a database dump from a Restic snapshot.
2. Comparing the original and restored dump using SHA-256.
3. Confirming that the hashes matched.
4. Importing the restored dump into an isolated PostgreSQL test container.
5. Confirming database import success.
6. Removing the temporary test environment.

The test container used a dedicated temporary volume, no published host ports and no network connectivity.

The live database was not used as the restore target.

The result supports a successful database dump restore and import under the recorded conditions.

It does not establish full Immich application recovery.

## 6. Storage incident evidence

The earlier backup failure was investigated using Windows system events and a later filesystem scan.

The investigation recorded storage-related Windows events during the failed backup attempt.

A later filesystem scan found no filesystem problems and no bad sectors. Subsequent backup attempts succeeded.

The original physical cause was not identified.

The portfolio therefore describes the incident as an unresolved storage-related failure rather than a confirmed disk hardware fault.

## 7. Limitations

The evidence has several limitations:

- Historical output does not prove current service state.
- A successful snapshot does not prove every file can be restored into a working application.
- A successful data-pack read does not explain every repository metadata observation.
- A successful PostgreSQL import does not prove application-level recovery.
- A configured network route does not establish end-to-end connectivity.
- A published container port does not prove public internet exposure.
- A read-only review is not a penetration test.
- Custom secret-pattern checks are not equivalent to a dedicated full-history secret scanner.

## 8. Reproducibility

Where safe and appropriate, test procedures should be reproducible without exposing private infrastructure details.

Examples include:

- Restic integrity checks against a controlled repository.
- Hash comparisons of original and restored files.
- Database imports into isolated test instances.
- Mermaid diagram validation.
- Markdown link checks.

Any future test should record:

- Date and time.
- Software version.
- Relevant test conditions.
- Expected result.
- Actual result.
- Exit code or error output, where applicable.
- Any limitations.

## 9. Reporting standard

Portfolio claims should be specific and bounded.

Prefer:

> The database dump was restored and imported successfully into an isolated PostgreSQL test environment.

Avoid:

> The entire application has been fully recovered.

Prefer:

> The full Restic data-pack read completed with no reported errors.

Avoid:

> The backup system is guaranteed to recover everything.

The purpose of evidence-led documentation is to show exactly what was demonstrated, not to imply more than the tests support.

