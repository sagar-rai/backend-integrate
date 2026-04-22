# Claude Code Plugin — backend-integrate

This directory contains the Claude Code plugin for `backend-integrate`. It is a complete, self-contained plugin that can be installed directly from this repo.

## Install

```
# Add this repo as a marketplace (one-time)
/plugin marketplace add sagar-rai/backend-integrate

# Install the plugin
/plugin install backend-integrate@backend-integrate
```

## Local development

```bash
claude --plugin-dir ./platforms/claude
```

Then reload after changes:
```
/reload-plugins
```

## Plugin structure

```
platforms/claude/
├── .claude-plugin/
│   └── plugin.json                  ← Plugin manifest (v2.0.0)
├── skills/
│   └── backend-integrate/
│       └── SKILL.md                 ← Skill: /backend-integrate:backend-integrate
├── agents/
│   └── backend-integrate.md         ← Orchestrator with inline A→B→C execution
└── README.md                        ← This file
```

## How to invoke

Invoke the skill explicitly or just describe what you want:

```
/backend-integrate:backend-integrate
```

Or naturally:

```
Integrate the payment service from github.com/acme/payment-svc — use INTEGRATION.md for context
```

## Execution tracks

After you approve the integration plan, the orchestrator works through three sequential tracks:

| Track | Starts when | Builds |
|---|---|---|
| A | Immediately after approval | Downstream HTTP/gRPC client + config struct + `.env.example` |
| B | After Track A completes | Service layer + dependency injection wiring |
| C | After Track B completes | Unit tests + integration tests + doc updates |

## Shared prompts

All behavioral prompts live in `prompts/` at the repo root and are shared between the Copilot and Claude plugins:

| File | Controls |
|---|---|
| `prompts/context_discovery.md` | How to fetch files from downstream repos via `gh` CLI |
| `prompts/integration_analysis.md` | How to synthesize fetched context |
| `prompts/clarification_guide.md` | 7 required questions before any plan is generated |
| `prompts/fleet_decomposition.md` | How to split integration into parallel tracks |

## Session cleanup

Temporary files are stored in `~/.agents/session/<uuid>/` and deleted automatically after context is loaded into memory. Nothing is written to your repo until you approve the plan.

## Requirements

- Claude Code installed and authenticated
- `gh` CLI installed and authenticated (`gh auth login`) — or let the agent install it for you
