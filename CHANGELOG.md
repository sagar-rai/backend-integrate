# Changelog

All notable changes to this project will be documented in this file.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.0.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## Versioning Policy

| Bump | When to use |
|---|---|
| **Patch** `1.0.x` | Prompt wording fixes, typo corrections, minor clarification tweaks |
| **Minor** `1.x.0` | New language/framework patterns, new file types discovered, new examples |
| **Major** `x.0.0` | Breaking changes to agent protocol, new platform support (e.g. Claude), structural reorganization |

---

## [Unreleased]

---

## [1.0.0] — 2026-04-17

### Added
- Initial release of `backend-integrate` as a GitHub Copilot CLI plugin
- Agent protocol (`backend-integrate.agent.md`) with 10-step integration workflow
- Context discovery (`prompts/context_discovery.md`) — fetches relevant files from downstream repos via `gh api`
- Integration analysis (`prompts/integration_analysis.md`) — synthesizes fetched files into actionable context
- Clarification guide (`prompts/clarification_guide.md`) — 7 required questions before any plan is generated
- Fleet decomposition (`prompts/fleet_decomposition.md`) — splits integration into parallel Track A/B/C tasks
- Language patterns for Go, Java, Node.js, Python, and Ruby
- `examples/INTEGRATION.md` — template for downstream service owners
- `docs/writing-integration-context.md` — guide for service owners on writing integration context files
- Zero-dependency design: works with only `gh auth login`
- Session workspace convention: `~/.copilot/sessions/<uuid>/` with guaranteed cleanup

---

## [2.0.0] — Planned (Phase 2)

### Added
- Claude plugin support (`platforms/claude/`) — Claude-compatible manifest and skill definition
- Shared `prompts/` directory used by both Copilot and Claude platforms
- `platforms/copilot/` — Copilot-specific manifest alongside Claude manifest

### Notes
- Phase 2 introduces the `platforms/` directory structure
- All prompt behavior remains shared across platforms — only manifests differ
- Existing Copilot plugin paths remain unchanged for backward compatibility

---

[Unreleased]: https://github.com/sagar-rai/backend-integrate-copilot/compare/v1.0.0...HEAD
[1.0.0]: https://github.com/sagar-rai/backend-integrate-copilot/releases/tag/v1.0.0
[2.0.0]: https://github.com/sagar-rai/backend-integrate-copilot/releases/tag/v2.0.0
