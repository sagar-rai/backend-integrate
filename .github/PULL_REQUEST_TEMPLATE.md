## What does this PR do?

<!-- Describe what changed and why. If you edited a prompt, include a before/after example. -->

## Type of change

- [ ] Prompt improvement (`prompts/`)
- [ ] New language/framework pattern (`fleet_decomposition.md`)
- [ ] New example (`examples/`)
- [ ] Documentation (`README.md`, `examples/`)
- [ ] Bug fix
- [ ] New feature
- [ ] Platform support (Copilot / Claude)

## Which downstream repo did you test against?

<!-- e.g. github.com/stripe/stripe-go -->

## Test checklist

- [ ] All 7 required questions from `clarification_guide.md` are asked before planning
- [ ] No code is written before the plan is presented and approved
- [ ] `gh` CLI is used for all GitHub access (no curl, no MCP)
- [ ] Temp files appear in `~/.agents/session/<uuid>/`
- [ ] Temp files are deleted after context is loaded
- [ ] The generated plan uses exact file paths (no placeholders)
- [ ] Fleet tracks respect A → B → C dependency order
- [ ] `plugin.json` version is bumped if behavior changed (see [CHANGELOG.md](../CHANGELOG.md))

## Breaking changes?

<!-- Does this change the agent protocol, restructure files, or require users to update anything? -->

## Related issues

<!-- Closes #... -->
