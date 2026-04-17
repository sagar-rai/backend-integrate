# backend-integrate

**Give Copilot the full context it needs to integrate any downstream service — automatically.**

One command. Copilot fetches the right files, asks the right questions, and builds the integration in parallel.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![GitHub Copilot](https://img.shields.io/badge/GitHub-Copilot%20Plugin-black?logo=github)](https://github.com/features/copilot)

</div>

---

## The problem

Integrating a downstream backend service is one of the most context-heavy tasks a developer faces. Before writing a single line of code, you need to understand:

- What endpoints exist and what they accept
- How authentication works (API key? OAuth? mTLS?)
- What environment variables to add
- What the request/response shapes look like
- How errors are structured and which ones to retry
- What patterns your own codebase expects

Then you need to somehow convey ALL of that to Copilot in a chat window — or just wing it and fix the mistakes.

`backend-integrate` solves this. Point it at the downstream service's GitHub repo, tell it which integration guide to read, and it handles the rest.

---

## Install

Install directly from GitHub:

```bash
copilot plugin install sagar-rai/backend-integrate
```

That's it. Requires `gh auth login` to be active.

> Once listed on [awesome-copilot](https://github.com/github/awesome-copilot), you'll also be able to install with:
> ```bash
> copilot plugin install backend-integrate@awesome-copilot
> ```

---

## Quick start

Just describe what you want to integrate:

```
Integrate the payment service from github.com/acme/payment-svc — use INTEGRATION.md for context, I need to add checkout support
```

```
I need to call the notification service (github.com/org/notify-svc, context file: DOWNSTREAM.md) to send order confirmation emails
```

```
Add support for the inventory service — repo is github.com/org/inventory, integration guide is docs/INTEGRATION.md
```

The more specific your command, the more precise the integration. Here's a real-world example integrating a gRPC service — pointing at a specific RPC, a specific client class to update, and a specific repository pattern to follow:

```
use docs/billing-engine-grpc-integration-guide.md from acme/billing-engine repo to integrate
ListPaymentMethods rpc from ProductAPI service — integrate the call in
BillingEngineClient, update the billing-engine-client version in build.sbt to the latest,
and see other methods to see how this call is being called from the repository layer in the project
and integrate a similar refresh in BillingEngineRepository
```

Copilot will take it from there.

---

## How it works

```
1. FETCH      gh CLI downloads the integration context .md + scans the repo
              for protos, OpenAPI specs, client examples, env config, models
                │
                ▼
2. ANALYZE    Copilot reads all files and synthesizes the integration surface:
              endpoints, auth mechanism, required config, data models
                │
                ▼
3. CLARIFY    Copilot asks you 7 questions about your service before planning:
              language/framework, patterns, error handling, tests, config
                │
                ▼
4. PLAN       A file-level integration plan is presented for your approval
                │
                ▼
5. BUILD      Parallel fleet agents implement the integration simultaneously:
              client → service layer → config → DI wiring → tests → docs
                │
                ▼
6. CLEAN UP   All temporary files removed from ~/.copilot/sessions/<uuid>/
```

---

## What Copilot will ask you

Before generating any code, Copilot asks 7 required questions to ensure the integration fits your codebase:

1. **Language & framework** — Go+Gin, Java+Spring Boot, Node+Express, Python+FastAPI, etc.
2. **Scope** — which specific endpoints/features you need (confirmed against what was found)
3. **Existing patterns** — do you have an HTTP client wrapper or gRPC client to follow?
4. **Error handling** — propagate up, fallback values, retry with backoff?
5. **Tests** — unit tests, integration tests, which framework?
6. **Architecture** — which service/repository layer should the integration live in?
7. **Config management** — env vars, config files, Vault, AWS SSM?

These questions prevent the most common integration mistakes: wrong error handling, mismatched patterns, missing config, untested code.

---

## What gets built

A typical integration produces:

| Track | What's created | When |
|---|---|---|
| A (immediate) | Downstream HTTP/gRPC client + config struct | Starts immediately |
| A (parallel) | `.env.example` updates + config registration | Starts immediately |
| B (after A) | Service layer + dependency injection wiring | After client exists |
| C (after B) | Unit tests + integration tests + doc updates | After service layer exists |

All tracks within a stage run in parallel. The total time is the sum of the critical path, not all tasks combined.

---

## Writing a good INTEGRATION.md

The plugin works best when the downstream service has a well-written integration context file. See [`docs/writing-integration-context.md`](docs/writing-integration-context.md) for a full guide, and [`examples/INTEGRATION.md`](examples/INTEGRATION.md) for a copy-paste template.

A great `INTEGRATION.md` includes:
- One-paragraph overview
- Working `curl` quick start
- Auth mechanism with exact env var names
- Config table (all required + optional vars)
- Key endpoints with request/response shapes
- Data models as TypeScript interfaces
- Retryable vs non-retryable error table
- Rate limits and retry strategy
- Client examples in common languages

---

## Fleet mode

`backend-integrate` uses Copilot's fleet (parallel agent) execution to implement multiple parts of an integration simultaneously. This means:

- The client and config updates start at the same time
- Tests start as soon as the service layer is ready
- Documentation updates run in parallel with tests
- You're not waiting for each file to be written one at a time

For a typical 10-file integration, fleet mode is significantly faster than sequential execution.

---

## Temporary files and cleanup

All files fetched from the downstream repo are stored temporarily in:

```
~/.copilot/sessions/<uuid>/
```

They are deleted automatically once Copilot has read them into its working context. Nothing is written to your repo until you approve the integration plan.

---

## Requirements

### GitHub CLI (`gh`)

The plugin uses `gh` for all GitHub access. **If `gh` is not installed, the agent will detect your OS and attempt to install it automatically.** To install it yourself:

<details>
<summary><strong>macOS</strong></summary>

```bash
brew install gh
```

No Homebrew?
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
brew install gh
```
</details>

<details>
<summary><strong>Linux (Debian / Ubuntu)</strong></summary>

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
<summary><strong>Linux (Fedora / RHEL / CentOS)</strong></summary>

```bash
dnf install 'dnf-command(config-manager)' -y
dnf config-manager --add-repo https://cli.github.com/packages/rpm/gh-cli.repo
dnf install gh --repo gh-cli -y
```
</details>

<details>
<summary><strong>Linux (Arch)</strong></summary>

```bash
pacman -S github-cli
```
</details>

<details>
<summary><strong>Windows</strong></summary>

**winget** (Windows 10 1709+ / Windows 11 — recommended):
```powershell
winget install --id GitHub.cli
```

**Chocolatey:**
```powershell
choco install gh
```

**Scoop:**
```powershell
scoop install gh
```

**MSI installer:** [github.com/cli/cli/releases/latest](https://github.com/cli/cli/releases/latest)
</details>

After installing:
```bash
gh auth login   # follow prompts: GitHub.com → HTTPS → authenticate via browser
gh auth status  # confirm it worked
```

### Other requirements

| Requirement | Details |
|---|---|
| **GitHub Copilot CLI** | The plugin runs inside Copilot CLI |
| **Downstream repo access** | Your `gh` token must have read access to the downstream repo |

No Python. No `pip install`. No npm. No Docker.

---

## Privacy

- **No telemetry** — the plugin never phones home
- **No MCP server** — all GitHub access via your own `gh` token, no context pollution
- **Temporary files only** — downloaded to `~/.copilot/sessions/<uuid>/` and deleted after use
- **No credentials written to disk** — API keys stay in env vars, never in downloaded files
- **Nothing written to your repo** — until you explicitly approve the integration plan

---

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for how to improve the plugin.

Contributions we love:
- Better file discovery patterns in `prompts/context_discovery.md`
- New language/framework patterns in `prompts/fleet_decomposition.md`
- Additional clarifying questions in `prompts/clarification_guide.md`
- New sections in `examples/INTEGRATION.md`
- Documentation improvements

---

## License

MIT — see [LICENSE](LICENSE)

---

> ⭐ **If this saves you time, consider starring the repo** — it helps other developers discover it.
