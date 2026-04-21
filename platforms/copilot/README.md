# Copilot Platform

This directory mirrors the GitHub Copilot plugin manifest for parity with the `claude/` directory introduced in Phase 2.

The authoritative Copilot manifest lives at:

```
.github/plugin/plugin.json
```

That path is what the Copilot CLI reads and **must not move**. This directory exists only for organizational symmetry with `platforms/claude/` — do not duplicate or replace `plugin.json` here.

**Note:** As of v2.0.0, the session workspace path changed from `~/.copilot/sessions/<uuid>/` to `~/.agents/session/<uuid>/` — a platform-neutral path shared by both Copilot and Claude plugins.
