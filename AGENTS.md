# AGENTS.md — Instructions for Copilot Agents Working on backend-integrate

This file tells any Copilot agent (coding agent, fleet sub-agent, or interactive session) exactly how to understand, navigate, and modify the `backend-integrate-copilot` codebase.

---

## What this repo is

`backend-integrate` is a **zero-dependency GitHub Copilot CLI plugin**. It helps developers integrate downstream backend services by fetching context from GitHub repos via `gh` CLI, asking clarifying questions, and executing integrations in parallel fleet mode.

**There is no executable code in this repo.** All behavior is encoded in markdown files — prompts, skill definitions, and agent instructions. Copilot reads these files and follows them. Improving the plugin means improving the markdown.

---

## Repo structure — what each file does

```
backend-integrate-copilot/
│
├── AGENTS.md                          ← YOU ARE HERE — agent instructions for this repo
│
├── .github/
│   ├── agents/
│   │   └── backend-integrate.agent.md ← PRIMARY: full agent protocol when plugin is invoked
│   ├── plugin/
│   │   └── plugin.json                ← Plugin manifest (do not edit paths without updating files)
│   └── copilot-instructions.md        ← Scoped instructions (globs: *.md, *.json)
│
├── skills/
│   └── backend-integrate/
│       └── SKILL.md                   ← Skill entry point with YAML frontmatter
│
├── prompts/                           ← BEHAVIORAL ENGINE — edit these to change plugin behavior
│   ├── context_discovery.md           ← HOW to fetch files from GitHub repos via gh CLI
│   ├── integration_analysis.md        ← HOW to read and synthesize downloaded context
│   ├── clarification_guide.md         ← WHAT questions to ask before planning (7 required)
│   └── fleet_decomposition.md         ← HOW to split integration into parallel fleet tracks
│
├── examples/
│   └── INTEGRATION.md                 ← Template for downstream service owners
│
├── docs/
│   └── writing-integration-context.md ← Guide for service owners writing INTEGRATION.md files
│
├── SKILL.md                           ← Root-level skill (mirrors skills/backend-integrate/SKILL.md)
├── README.md                          ← User-facing documentation
├── CONTRIBUTING.md                    ← Contributor guide
├── LICENSE                            ← MIT
├── CODE_OF_CONDUCT.md
├── SECURITY.md
└── .gitignore
```

---

## The most important files to know

When working on this repo, these are the files that matter most:

### 1. `.github/agents/backend-integrate.agent.md`
The full execution protocol for the plugin. When a user invokes the agent, Copilot reads this file and follows it step by step. **Editing this file changes how the plugin behaves end-to-end.** Structure: YAML frontmatter (`name`, `description`) + numbered steps 0–9.

### 2. `prompts/` directory
Four files that control specific phases of the plugin:

| File | Phase | Key rule |
|---|---|---|
| `context_discovery.md` | Fetching files | Always use `gh api`, never curl or MCP |
| `integration_analysis.md` | Analyzing context | Read CONTEXT_FILE first, then README, then specs |
| `clarification_guide.md` | Asking questions | All 7 required questions MUST fire before planning |
| `fleet_decomposition.md` | Building in parallel | Track A → Track B → Track C dependency order |

### 3. `skills/backend-integrate/SKILL.md`
The skill definition. Must have YAML frontmatter with `name` and `description`. The `description` field is what Copilot uses to decide when to invoke this skill — keep it accurate and keyword-rich.

### 4. `.github/plugin/plugin.json`
The plugin manifest. The `agents` and `skills` arrays must reference paths that actually exist on disk. Never change a path here without moving the corresponding file.

---

## Hard rules — never violate these

1. **No executable code** — no Python, no shell scripts, no JavaScript, no binaries
2. **No MCP server** — all GitHub access via `gh api` or `gh repo clone` only
3. **No external dependencies** — zero `pip install`, `npm install`, `apt-get` (except `gh` itself)
4. **No hardcoded credentials** — no API keys, tokens, passwords, or personal identifiers
5. **No external data uploads** — the plugin must never send data to third-party services
6. **Clarification gate must stay** — never allow planning before all 7 required questions are answered
7. **Plan gate must stay** — never allow code generation before the user approves the plan
8. **Cleanup must always run** — `rm -rf ~/.copilot/sessions/<uuid>/` must always happen

---

## How to make common changes

### Add a new file type to discover (e.g., AsyncAPI specs)

Edit `prompts/context_discovery.md`:
- Find the `grep -iE` filter pattern in Step 5
- Add your pattern (e.g., `asyncapi\.(yaml|yml|json)$`)
- Add a row to the **File type priority** table explaining why it's useful

### Add a new clarifying question

Edit `prompts/clarification_guide.md`:
- Add it under **Required questions** (if always needed) or **Optional questions** (if context-dependent)
- Follow the format: `### N. Topic`, bold question text, explanation of why it matters
- Update the post-answer summary template at the bottom to include the new question

### Add a new language/framework pattern

Edit `prompts/fleet_decomposition.md`:
- Find the **Language-specific patterns** section
- Add a new `### Language (Framework)` block with: Client, Service layer, DI, Tests entries
- Follow the exact same format as the existing entries (Go, Java, Node.js, Python, Ruby)

### Change the agent's step-by-step protocol

Edit `.github/agents/backend-integrate.agent.md`:
- Steps are numbered 0–9
- Step 0 = `gh` install/auth check (do not remove)
- Steps 3–4 = file fetching (must use `gh api`, not curl/MCP)
- Step 6 = clarifying questions (gate must remain)
- Step 7 = plan presentation (gate must remain)
- Step 9 = cleanup (must always run last)

### Update the plugin version

Edit `.github/plugin/plugin.json`:
- Bump `version` (semver: `1.0.0` → `1.1.0` for behavior changes, `1.0.1` for doc fixes)
- Do not change `agents` or `skills` paths unless you've also moved those files

### Add a new example

Add a file to `examples/` (e.g., `examples/INTEGRATION-grpc.md`):
- Follow the structure of `examples/INTEGRATION.md`

---

## How to test your changes

There is no automated test suite. Test manually:

### Basic test (30 seconds)
```bash
cd ~/backend-integrate-copilot
copilot
```
Then invoke:
```
Integrate github.com/stripe/stripe-go using README.md as context — I need charge support
```

### Full checklist
- [ ] `gh` is detected and auto-install triggers correctly if missing
- [ ] The integration context file is downloaded to `~/.copilot/sessions/<uuid>/`
- [ ] Relevant files (proto, openapi, client, env) are downloaded — not everything
- [ ] An analysis summary is shown to the user before questions begin
- [ ] All 7 required questions from `clarification_guide.md` are asked
- [ ] No plan is generated until all questions are answered
- [ ] The plan shows exact file paths (not placeholders)
- [ ] The user is asked to approve the plan before any code is written
- [ ] Fleet tracks follow A → B → C dependency order
- [ ] `~/.copilot/sessions/<uuid>/` is deleted after context is loaded
- [ ] No MCP calls are made — only `gh` CLI

### Good public repos to test against

| Repo | Tests |
|---|---|
| `github.com/stripe/stripe-go` | REST client, good README |
| `github.com/googleapis/go-genai` | gRPC, proto files |
| Any repo with an `INTEGRATION.md` | Happy path with context file |
| Any repo without an `INTEGRATION.md` | Fallback path — scan + filter |

---

## What NOT to generate when working on this repo

When a Copilot agent is asked to improve or extend this plugin, it must NOT:

- Create `.py`, `.js`, `.ts`, `.go`, `.sh`, or any other executable files
- Add a `requirements.txt`, `package.json`, `go.mod`, or `Makefile`
- Add MCP server configs to `.github/` or anywhere else
- Introduce `curl` as an alternative to `gh api`
- Remove the two plan/clarify gates
- Remove the session cleanup step
- Hardcode any personal data, repo paths, usernames, or tokens
- Generate a test suite (there is none by design — it's a prompt-driven skill)

---

## YAML frontmatter requirements

Two files require YAML frontmatter and will break the plugin if it's missing or wrong:

**`SKILL.md` and `skills/backend-integrate/SKILL.md`:**
```yaml
---
name: backend-integrate
description: "Fetch downstream service context from a GitHub repo and execute a backend integration into your current service — fetches relevant files via gh CLI, asks clarifying questions, creates a plan, and runs it in fleet mode. Use when a developer wants to integrate or call a downstream service into their backend."
---
```

**`.github/agents/backend-integrate.agent.md`:**
```yaml
---
name: backend-integrate
description: "Fetch downstream service context from a GitHub repo and execute a backend integration — uses gh CLI to get files, asks clarifying questions, creates a parallel integration plan, and executes in fleet mode. Use when a developer wants to integrate a downstream or external service into their backend."
---
```

The `description` field is used by Copilot to match user intent to the right skill/agent. Keep it specific, action-oriented, and keyword-rich.

---

## Session workspace convention

Whenever the plugin downloads files from a downstream repo, it uses:

```
~/.copilot/sessions/<uuid>/
```

- UUID is generated with `uuidgen | tr '[:upper:]' '[:lower:]'`
- Files are written here during context fetch, read into Copilot's memory, then deleted
- The directory is always cleaned up with `rm -rf "$SESSION"` — never leave it behind
- On Windows, the equivalent is `$env:USERPROFILE\.copilot\sessions\<uuid>\`

If you add new file-fetching steps to the agent, always write to `$SESSION/` and always clean up.
