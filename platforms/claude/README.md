# Claude Plugin Support — Phase 2

This directory will contain the Claude plugin manifest and skill definition for Phase 2 of `backend-integrate`.

## Planned structure

```
platforms/
  claude/
    manifest.json          ← Claude plugin manifest (name, description, skills)
    skills/
      backend-integrate/
        SKILL.md           ← Claude-compatible skill definition with frontmatter
  copilot/
    plugin.json            ← (symlink or copy of .github/plugin/plugin.json for parity)
```

## Design principles for Phase 2

- **Shared prompts** — `prompts/` is platform-agnostic. Both Copilot and Claude read the same `context_discovery.md`, `integration_analysis.md`, `clarification_guide.md`, and `fleet_decomposition.md`.
- **Platform-specific manifests only** — each platform gets its own manifest format; no prompt duplication.
- **Same version, both platforms** — `plugin.json` and `manifest.json` share the same semver version. A release bumps both.
- **Backward compatible** — the Copilot plugin path (`.github/plugin/plugin.json`) will not move in Phase 2.

## Version bump

Phase 2 will ship as `v2.0.0` — a major version bump because it introduces a new platform directory and manifest format. See [CHANGELOG.md](../../CHANGELOG.md).

## Contributing

If you want to help build Phase 2 Claude support, open a feature request using the [Claude plugin support](../../.github/ISSUE_TEMPLATE/feature_request.yml) template and tag it `enhancement`.
