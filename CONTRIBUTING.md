# Contributing to backend-integrate

Thank you for your interest in contributing! This plugin helps developers integrate downstream backend services by giving Copilot the full context it needs. Since the plugin is entirely markdown-based (no code, no dependencies), contributions are accessible to anyone who can write clear, structured documentation.

---

## How the plugin works

Before contributing, understand the architecture:

```
.github/agents/backend-integrate.agent.md  ← What the agent does when invoked
.github/plugin/plugin.json                 ← Plugin manifest (don't edit lightly)
skills/backend-integrate/SKILL.md          ← How the skill is described to Copilot
prompts/context_discovery.md               ← HOW to fetch files from GitHub repos
prompts/integration_analysis.md            ← HOW to synthesize downloaded context
prompts/clarification_guide.md             ← WHAT questions to ask before planning
prompts/fleet_decomposition.md             ← HOW to parallelize the integration
examples/INTEGRATION.md                    ← Template for downstream service owners
docs/writing-integration-context.md        ← Guide for service owners writing INTEGRATION.md files
```

All behavior is driven by the markdown in `prompts/`. No code runs — Copilot reads these files and follows the instructions.

---

## How to contribute

1. **Fork the repo** and create a feature branch
2. **Make your changes** — see accepted contributions below
3. **Test manually** — invoke the skill against a real public GitHub repo
4. **Submit a PR** with:
   - What you changed and why
   - A before/after example if you changed a prompt
   - Which downstream repo you tested against

---

## What we accept

### Prompt improvements (`prompts/`)

This is the most impactful area. Better prompts → better integrations.

| File | What to improve |
|---|---|
| `context_discovery.md` | Add new file patterns (AsyncAPI, GraphQL, Terraform, k8s manifests), improve gh CLI commands, better fallback handling |
| `integration_analysis.md` | Better synthesis strategies, new extraction targets (event schemas, SLAs), improve incomplete-context handling |
| `clarification_guide.md` | New required or optional questions, better question wording, framework-specific follow-ups |
| `fleet_decomposition.md` | New language/framework patterns (Rust+Axum, Kotlin+Ktor, PHP+Laravel), better track structures |

When editing prompts:
- Keep language clear and imperative ("Do X", "Ask Y", "Never Z")
- Include concrete examples and bash commands where relevant
- Test that the clarification gate still fires before any planning
- Verify cleanup still runs at the end

### New language patterns

If your language/framework isn't in `prompts/fleet_decomposition.md`, add it. Follow the existing format:

```markdown
### {Language} ({Framework})
- Client: {how clients are structured}
- Service layer: {how service layer is structured}
- DI: {dependency injection approach}
- Tests: {testing framework and patterns}
```

### Examples (`examples/`)

New or improved integration context templates for specific API styles:
- gRPC-specific `INTEGRATION.md` template
- GraphQL-specific template
- AsyncAPI / event-driven template
- Webhook receiver template

### Documentation (`docs/`)

- Improvements to `writing-integration-context.md` — better guidance for specific API styles, new sections for emerging patterns (AsyncAPI, GraphQL subscriptions, etc.)

### Bug reports

If the skill produces wrong output, open an issue with:
- The downstream repo URL (or a minimal reproduction)
- What context file you used
- What output Copilot generated
- What the correct output should have been

---

## What we don't accept

| What | Why |
|---|---|
| Python scripts or any executable code | The plugin is zero-dependency by design |
| MCP server integrations | Causes context pollution — explicitly out of scope |
| External dependencies (`pip install`, `npm install`, etc.) | Users should need nothing beyond `gh auth login` |
| Hardcoded credentials, tokens, or personal data | Security policy |
| Changes that upload data to external services | Privacy policy |
| Modifications to `plugin.json` agent/skill paths | Without corresponding file changes — breaks the plugin |

---

## Testing your changes

There is no automated test suite — this is a prompt-driven skill. Test manually:

### Basic test

1. Make your changes
2. Open Copilot CLI and invoke the skill:
   ```
   Integrate github.com/stripe/stripe-go using their README as context
   ```
3. Verify Copilot:
   - Downloads relevant files using `gh` CLI commands
   - Presents an analysis summary
   - Asks all 7 required clarifying questions before planning
   - Produces a file-level plan before writing any code
   - Cleans up `~/.copilot/sessions/<uuid>/` after context is loaded

### Test checklist

- [ ] All 7 required questions from `clarification_guide.md` are asked
- [ ] No code is written before the plan is presented
- [ ] No code is written before the plan is approved
- [ ] `gh` CLI is used for all GitHub access (no curl, no MCP)
- [ ] Temp files appear in `~/.copilot/sessions/<uuid>/`
- [ ] Temp files are deleted after context is loaded
- [ ] The generated plan uses exact file paths (not placeholders)
- [ ] Fleet tracks match the dependency order (A → B → C)

### Test repos

Good public repos to test against:

| Repo | Why useful |
|---|---|
| `github.com/stripe/stripe-go` | Real REST client library, good README |
| `github.com/grpc/grpc-go` | gRPC, has proto files |
| `github.com/open-telemetry/opentelemetry-go` | Complex integration surface |
| Any repo with an `INTEGRATION.md` | Tests the primary happy path |

---

## Prompt editing principles

1. **Be imperative** — "Fetch the file", not "You could fetch the file"
2. **Be concrete** — include actual bash commands, not just descriptions
3. **Preserve the gates** — clarify-before-plan and plan-before-execute are non-negotiable
4. **Keep cleanup** — `rm -rf "$SESSION"` must always happen
5. **No MCP** — all GitHub access via `gh api` or `gh repo clone`

---

## Editing `plugin.json`

Only edit `plugin.json` if you are:
- Changing the plugin version (bump for any user-visible behavior change)
- Adding a new agent or skill file (and creating the corresponding file)
- Updating author or repo metadata

Never change the `agents` or `skills` paths without creating/moving the referenced files.

---

## License

By contributing, you agree your contributions will be licensed under the [MIT License](LICENSE).
