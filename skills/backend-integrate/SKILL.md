---
name: backend-integrate
description: "Fetch downstream service context from a GitHub repo and execute a backend integration into your current service — fetches relevant files via gh CLI, asks clarifying questions, creates a plan, and runs it in fleet mode. Use when a developer wants to integrate or call a downstream service into their backend."
---

# backend-integrate Skill

## What this skill does

When invoked, this skill fetches context about a downstream service from its GitHub repository using the `gh` CLI, analyzes the integration surface, asks the developer clarifying questions, then creates and executes an integration plan in parallel fleet mode.

## Invocation

Run this skill when the user says something like:
- "Integrate the X service from github.com/org/repo"
- "I need to add calls to the Y service, use INTEGRATION.md for context"
- "Add downstream service Z to my current service"
- "Help me integrate github.com/org/repo using their DOWNSTREAM.md guide"

## Step-by-step execution

### Step 0 — Verify `gh` is installed and authenticated

Before anything else:

```bash
gh --version 2>/dev/null || echo "NOT_INSTALLED"
```

If `gh` is not installed, detect the OS and install it:

```bash
OS="$(uname -s 2>/dev/null || echo Windows)"
case "$OS" in
  Darwin)
    command -v brew &>/dev/null && brew install gh || {
      /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
      brew install gh
    }
    ;;
  Linux)
    if command -v apt-get &>/dev/null; then
      curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg \
        | dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
      chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg
      echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" \
        | tee /etc/apt/sources.list.d/github-cli.list > /dev/null
      apt-get update && apt-get install gh -y
    elif command -v dnf &>/dev/null; then
      dnf config-manager --add-repo https://cli.github.com/packages/rpm/gh-cli.repo
      dnf install gh --repo gh-cli -y
    elif command -v pacman &>/dev/null; then
      pacman -S github-cli --noconfirm
    else
      echo "Please install gh manually: https://github.com/cli/cli/blob/trunk/docs/install_linux.md"
      exit 1
    fi
    ;;
  Windows*)
    if command -v winget &>/dev/null; then winget install --id GitHub.cli -e
    elif command -v choco &>/dev/null; then choco install gh -y
    elif command -v scoop &>/dev/null; then scoop install gh
    else
      echo "Please install gh manually. Options:"
      echo "  winget install --id GitHub.cli"
      echo "  choco install gh"
      echo "  https://github.com/cli/cli/releases/latest"
      exit 1
    fi
    ;;
esac
```

Then check authentication:

```bash
gh auth status 2>/dev/null || gh auth login
```

If `gh auth login` is needed, guide the user: choose GitHub.com → HTTPS → authenticate via browser. Do not proceed until `gh auth status` succeeds.

### Step 1 — Parse inputs

Extract from the user's message:
- `REPO_URL`: the GitHub repo URL of the downstream service (e.g., `github.com/acme/payment-svc` → owner=`acme`, repo=`payment-svc`)
- `CONTEXT_FILE`: the `.md` file name containing integration context (e.g., `INTEGRATION.md`, `DOWNSTREAM.md`)
- `GOAL`: what the user wants to integrate or accomplish

### Step 2 — Create session directory

```bash
UUID=$(uuidgen | tr '[:upper:]' '[:lower:]')
SESSION="$HOME/.agents/session/$UUID"
mkdir -p "$SESSION"
```

### Step 3 — Fetch the integration context file

```bash
gh api repos/{owner}/{repo}/contents/{CONTEXT_FILE} --jq '.content' | base64 -d > "$SESSION/$CONTEXT_FILE"
```

If this fails (file not found), inform the user and ask for the correct file name.

### Step 4 — Discover and download relevant files

Get the repo's default branch and file tree:

```bash
# Try main, fall back to master
SHA=$(gh api repos/{owner}/{repo}/git/refs/heads/main --jq '.object.sha' 2>/dev/null || \
      gh api repos/{owner}/{repo}/git/refs/heads/master --jq '.object.sha')

# Get all file paths
gh api "repos/{owner}/{repo}/git/trees/$SHA?recursive=1" --jq '.tree[].path' \
  > "$SESSION/all_files.txt"
```

Filter for high-value integration files:

```bash
grep -iE '(README\.md|\.proto$|openapi\.(yaml|json)$|swagger\.(yaml|json)$|\.env\.example$|\.env\.sample$|config\.example|/api/|/handler/|/endpoint/|client\.|Client\.|/model/|/dto/|/schema/|Makefile$)' \
  "$SESSION/all_files.txt" | head -40 > "$SESSION/relevant_files.txt"
```

Download each relevant file:

```bash
while IFS= read -r filepath; do
  dir="$SESSION/$(dirname "$filepath")"
  mkdir -p "$dir"
  gh api "repos/{owner}/{repo}/contents/$filepath" --jq '.content' \
    | base64 -d > "$SESSION/$filepath" 2>/dev/null || true
done < "$SESSION/relevant_files.txt"
```

If relevant file count > 50 or user requests full repo, clone with shallow depth instead:

```bash
gh repo clone {owner}/{repo} "$SESSION/repo" -- --depth=1 --quiet
```

### Step 5 — Analyze context

Read all files in `$SESSION/`. Synthesize:
- What the downstream service does
- Which endpoints/methods are needed for the stated goal
- Authentication mechanism (API key, Bearer token, OAuth, mTLS, etc.)
- Required environment variables / config
- Key data models / request-response shapes
- Any rate limits, retry hints, or circuit breaker recommendations from the docs

### Step 6 — Ask clarifying questions

Before writing any plan or code, ask the user (one at a time or as a grouped form):

**Required:**
1. What language and framework is your current service? (e.g., Go+Gin, Java+Spring Boot, Node+Express, Python+FastAPI, Ruby+Rails)
2. Which specific endpoints/features of the downstream service do you need? (confirm against what was found)
3. Do you have an existing HTTP client wrapper or gRPC client pattern to follow?
4. How should errors be handled? (propagate up, fallback values, retry with backoff)
5. Do you need unit tests? Integration tests? Which testing framework?
6. Are there service/repository layer patterns in your codebase to follow?
7. How is configuration managed? (env vars, config file, secrets manager like Vault/AWS SSM)

**Ask if ambiguous:**
- Is there a circuit breaker or retry library already in use?
- Do you need OpenTelemetry/tracing on the new client?
- Synchronous (HTTP/gRPC) or async (message queue) integration?

Do NOT proceed to planning until all required questions are answered.

### Step 7 — Create and present integration plan

Using the answers and the synthesized context, create a file-level plan:
- List every file to be created or modified
- Group into parallel tracks (see `prompts/fleet_decomposition.md`)
- Present the plan to the user for approval before any code is written

### Step 8 — Execute in fleet mode

Once the user approves, launch parallel agents per the fleet decomposition:
- Agent 1: Create the downstream HTTP/gRPC client
- Agent 2 (after 1): Create the service layer wrapping the client
- Agent 3 (parallel with 1): Update config/env files
- Agent 4 (after 1): Wire into dependency injection
- Agent 5+ (after 2): Write tests and update docs

### Step 9 — Clean up session files

```bash
rm -rf "$SESSION"
```

Confirm cleanup to the user.
