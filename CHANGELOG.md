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

## [2.0.0] — 2026-04-21

### Added
- Claude Code plugin support (`platforms/claude/`) — full plugin with manifest, skill, and orchestrator agent
- `.claude-plugin/marketplace.json` at repo root — enables `/plugin marketplace add sagar-rai/backend-integrate` install flow
- `platforms/claude/.claude-plugin/plugin.json` — plugin manifest
- `platforms/claude/agents/backend-integrate.md` — orchestrator agent (model: sonnet, maxTurns: 60), protocol mirrors the Copilot agent
- `platforms/claude/skills/backend-integrate/SKILL.md` — Claude skill definition with full 9-step protocol inline, matching the Copilot SKILL.md format

### Changed
- Session workspace path migrated from `~/.copilot/sessions/<uuid>/` to `~/.agents/session/<uuid>/` — platform-neutral, shared by both Copilot and Claude plugins
- Updated all path references across: agent protocol, prompts, skills, README, CONTRIBUTING, AGENTS.md, SECURITY.md, copilot-instructions, PR template
- Integration execution consolidated to 3 tracks (A → B → C): Track A handles client + config + `.env.example`, Track B handles service layer + DI, Track C handles tests + docs

### Notes
- All `prompts/` files are shared across platforms — no duplication between Copilot and Claude
- Copilot plugin paths (`.github/plugin/`, `.github/agents/`) unchanged — fully backward compatible
- Claude Code install: `/plugin marketplace add sagar-rai/backend-integrate` then `/plugin install backend-integrate@backend-integrate`
- Install from feature branch: `/plugin marketplace add sagar-rai/backend-integrate#<branch-name>`
- Local testing: `claude --plugin-dir ./platforms/claude` from any project directory

---

[Unreleased]: https://github.com/sagar-rai/backend-integrate/compare/v2.0.0...HEAD
[1.0.0]: https://github.com/sagar-rai/backend-integrate/releases/tag/v1.0.0
[2.0.0]: https://github.com/sagar-rai/backend-integrate/releases/tag/v2.0.0
