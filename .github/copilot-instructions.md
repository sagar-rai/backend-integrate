---
description: "Instructions for working on the backend-integrate-copilot codebase — a zero-dependency Copilot CLI plugin for orchestrating downstream service integrations."
globs: "**/*.md,**/*.json"
---

> **For agents:** See `AGENTS.md` in the repo root for the full agent guide — file map, hard rules, how to make common changes, and the manual test checklist.

# backend-integrate Development Instructions

## What this repo is

A Copilot CLI plugin that helps developers integrate downstream backend services. It works entirely through skill and agent markdown files — there is no Python, no external dependencies, and no MCP server.

## Architecture

```
.github/
  agents/backend-integrate.agent.md  ← Full agent protocol (what to do when invoked)
  plugin/plugin.json                 ← Plugin manifest (name, version, skills, agents)
  copilot-instructions.md            ← This file
skills/
  backend-integrate/SKILL.md         ← Skill definition (frontmatter + invocation guide)
prompts/
  context_discovery.md               ← How to fetch and filter repo files via gh CLI
  integration_analysis.md            ← How to synthesize downloaded context
  clarification_guide.md             ← Required questions before planning
  fleet_decomposition.md             ← How to split integration into parallel fleet tasks
examples/
  INTEGRATION.md                     ← Template for downstream service owners
docs/
  writing-integration-context.md     ← Guide for writing good INTEGRATION.md files
SKILL.md                             ← Root-level skill (mirrors skills/backend-integrate/SKILL.md)
```

## Key conventions

- **No Python, no scripts** — all logic lives in markdown files as Copilot instructions
- **No MCP server** — all GitHub access goes through `gh` CLI commands
- **No external dependencies** — the plugin installs with zero setup beyond `gh auth login`
- **Prompts are plain markdown** — edit them directly without touching any code
- **Frontmatter is required** on SKILL.md files: `name` and `description` fields
- **plugin.json** must reference all agent and skill paths correctly

## When editing prompts

The `prompts/` directory contains the behavioral instructions Copilot follows:

| File | What it controls |
|---|---|
| `context_discovery.md` | Which files to fetch from downstream repos, gh CLI commands |
| `integration_analysis.md` | How to read and synthesize fetched files |
| `clarification_guide.md` | Required questions before generating any plan |
| `fleet_decomposition.md` | How to split integration work into parallel agent tracks |

When editing these, always verify that:
1. The clarifying questions are still all asked before planning
2. The gh CLI commands use `gh api` (not MCP, not curl)
3. Temp files still go to `~/.copilot/sessions/<uuid>/` and get cleaned up
4. No credentials or tokens are hardcoded

## Testing changes

There is no automated test suite — this is a prompt-driven skill.

To test manually:
1. Invoke the skill against a real public GitHub repo with a README
2. Verify all clarifying questions are asked before any plan is generated
3. Verify temp files are created in `~/.copilot/sessions/<uuid>/`
4. Verify temp files are cleaned up after the session
5. Confirm no external calls are made except via `gh` CLI

## What NOT to do

- Do not add Python scripts or any executable code
- Do not add MCP server configurations
- Do not hardcode usernames, tokens, repo paths, or personal data
- Do not add `pip install` or `npm install` requirements
- Do not modify plugin.json agent/skill paths without updating the actual files
