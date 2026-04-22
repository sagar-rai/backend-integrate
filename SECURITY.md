# Security Policy

## Reporting a Vulnerability

If you discover a security vulnerability in `backend-integrate`, please report it by opening a GitHub Issue with the label `security`.

**Do not include credentials, tokens, or sensitive data in issues or pull requests.**

## Security Principles

This plugin is designed with security in mind:
- **No external data uploads** — all file fetching uses your own `gh` CLI token
- **No MCP server** — avoids context pollution and unintended data sharing
- **Temporary files only** — context is downloaded to `~/.agents/session/<uuid>/` and deleted after use
- **No hardcoded credentials** — never commit tokens, API keys, or personal identifiers

## Supported Versions

We support the latest version of this plugin only.
