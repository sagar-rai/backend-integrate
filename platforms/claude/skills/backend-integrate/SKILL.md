---
name: backend-integrate
description: "Fetch downstream service context from a GitHub repo and execute a backend integration into your current service — fetches relevant files via gh CLI, asks clarifying questions, creates a plan, and runs it in fleet mode. Use when a developer wants to integrate or call a downstream service into their backend."
---

# backend-integrate — Downstream Service Integration Skill

This skill gives Claude everything it needs to plan and execute a backend service integration. Point it at a downstream service's GitHub repo, tell it which context file to read, and it fetches the right files, asks the right questions, and builds the integration for you using parallel fleet agents.

## Prerequisites

### GitHub CLI (`gh`)

The skill uses `gh` for all GitHub access. **If `gh` is not installed, the agent will detect your OS and install it automatically.** To install it yourself:

<details>
<summary><strong>macOS</strong></summary>

```bash
brew install gh
```

No Homebrew? Install it first:
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew install gh
```
</details>

<details>
<summary><strong>Linux (Debian/Ubuntu)</strong></summary>

```bash
type -p curl >/dev/null || apt-get install curl -y
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg \
  | dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] \
  https://cli.github.com/packages stable main" \
  | tee /etc/apt/sources.list.d/github-cli.list > /dev/null
apt-get update && apt-get install gh -y
```
</details>

<details>
<summary><strong>Windows</strong></summary>

```powershell
winget install --id GitHub.cli
```
</details>

After installing, authenticate:

```bash
gh auth login
```

---

## How to invoke

Invoke this skill as `/backend-integrate:backend-integrate`, or describe what you want to integrate naturally:

```
Integrate the payment service from github.com/acme/payment-svc — use INTEGRATION.md for context, I need to add checkout support
```

```
I need to call the notification service (github.com/org/notify-svc, context file: DOWNSTREAM.md) to send order confirmation emails
```

```
Add support for the inventory service — repo is github.com/org/inventory, integration guide is docs/INTEGRATION.md
```

## What Claude will do

1. **Fetch context** — downloads the specified `.md` context file and scans the repo for relevant files (proto definitions, OpenAPI specs, client examples, env config, data models) using `gh` CLI
2. **Analyze** — reads all fetched files and synthesizes the integration surface: endpoints, auth mechanism, required config, data models
3. **Ask clarifying questions** — confirms your language/framework, patterns, error handling approach, and test requirements before writing any code
4. **Create a plan** — produces a file-level integration plan broken into parallel tracks for review
5. **Execute in fleet mode** — spawns parallel subagents (`fleet-a`, `fleet-b`, `fleet-c`) to implement the client, service layer, config, DI wiring, and tests simultaneously
6. **Clean up** — removes all temporary files from `~/.agents/session/<uuid>/` automatically

## Fleet agents

| Agent | Track | What it builds |
|---|---|---|
| `fleet-a` | A (immediate) | Downstream HTTP/gRPC client + config struct |
| `fleet-b` | B (after A) | Service layer + dependency injection wiring |
| `fleet-c` | C (after B) | Unit tests + integration tests + doc updates |

## Session cleanup

All temporary files are stored in `~/.agents/session/<uuid>/` during the integration session and deleted automatically when context extraction is complete. Nothing is written to your repo until the plan is approved.
