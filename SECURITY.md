# Security Policy

## Reporting a security issue

Do not open a public issue containing credentials, private hostnames, access tokens, certificate material, IP addresses or details that would enable access to a live service.

For a private repository, use a private communication channel with the repository owner. For a public repository, enable GitHub private vulnerability reporting where available, or provide a private contact method in the repository settings.

## Repository hygiene

- Never commit `.env` files, password files, tokens, private keys, database files or raw logs.
- Review Caddy snippets, Compose files and screenshots for internal hostnames, access details and secret values before publishing.
- Keep raw command output in a private evidence location; commit only sanitised summaries.
- Rotate any credential that is accidentally committed. Removing it from the latest commit does not remove it from Git history.
