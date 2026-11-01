# Security Baseline

**Snapshot date:** 1 November 2026  
**Mode:** observation only; no firewall or container settings changed.

## Windows Firewall

Observed effective profile configuration:

| Profile | Enabled | Default inbound | Default outbound |
|---|---|---|---|
| Domain | Yes | Block | Allow |
| Private | Yes | Block | Allow |
| Public | Yes | Block | Allow |

The active Ethernet connection was classified as Private. A NordLynx interface was also classified as Private but reported `NoTraffic` at the time of inspection.

The default inbound block is a useful baseline. Explicit allow rules still create exceptions and must be considered separately.

## Historical firewall exception details omitted## Container-level concerns observed

[Historical container-security observation omitted; generalised guidance remains.]
[Historical container-security observation omitted; generalised guidance remains.]
[Historical container-security observation omitted; generalised guidance remains.]
[Historical container-security observation omitted; generalised guidance remains.]
5. **Moving image tags:** `latest`, `main`, `stable` and `release` tags can change over time. Record deployed digests and maintain a tested rollback path.
6. **Uneven health checks:** some containers have health checks and others do not. Running state alone is not an application health check.
7. **Secrets:** the Twingate connector and Open WebUI expose names of environment variables related to credentials/API endpoints. Values were not collected and must never be committed.

## Risk ranking (provisional)

| Priority | Finding | Why it matters | Safe next step |
|---|---|---|---|
| [Historical container finding] | [Specific exposure details omitted] | [General mitigation guidance retained] |
| [Historical container finding] | [Specific exposure details omitted] | [General mitigation guidance retained] |
| [Historical firewall finding] | [Specific rule scope omitted] | [General mitigation guidance retained] |
| Medium | Published application ports | Can bypass intended proxy path depending on firewall/network | Decide intended client networks, then test one service at a time |
| Medium | Root containers | Increases impact if application is compromised | Check image support and volume ownership before changing users |
| [Historical firewall finding] | [Specific rule scope omitted] | [General mitigation guidance retained] |
| [Historical firewall finding] | [Specific rule scope omitted] | [General mitigation guidance retained] |
| Low/Medium | Unresolved Kali Caddy route | Stale route or missing service | Identify intended service or document route as inactive |

## Scope limitation

This is not a penetration test, internet exposure test or complete Docker hardening audit. No external scanning, vulnerability scanning, credential review or packet-level reachability test was performed. A local host port binding does not prove internet accessibility.
