# Context Discovery — Fetching Downstream Service Files

This guide instructs Copilot on how to discover and download relevant files from a downstream service's GitHub repository using only the `gh` CLI.

## Inputs

- `OWNER`: GitHub organization or user (e.g., `acme`)
- `REPO`: Repository name (e.g., `payment-svc`)
- `CONTEXT_FILE`: The integration context markdown file (e.g., `INTEGRATION.md`)
- `UUID`: A unique session ID (generate with `uuidgen`)

## Step 1 — Create session workspace

```bash
UUID=$(uuidgen | tr '[:upper:]' '[:lower:]')
SESSION="$HOME/.copilot/sessions/$UUID"
mkdir -p "$SESSION"
echo "Session: $SESSION"
```

## Step 2 — Fetch the integration context file

This is the most important file — the downstream service owner wrote it to explain how to integrate.

```bash
gh api repos/$OWNER/$REPO/contents/$CONTEXT_FILE \
  --jq '.content' | base64 -d > "$SESSION/$CONTEXT_FILE"
```

**If this fails:** The file may not exist or may be in a subdirectory. Tell the user and ask them to confirm the path (e.g., `docs/INTEGRATION.md` vs `INTEGRATION.md`).

## Step 3 — Get the default branch SHA

```bash
SHA=$(gh api repos/$OWNER/$REPO/git/refs/heads/main \
  --jq '.object.sha' 2>/dev/null)

if [ -z "$SHA" ]; then
  SHA=$(gh api repos/$OWNER/$REPO/git/refs/heads/master \
    --jq '.object.sha' 2>/dev/null)
fi

if [ -z "$SHA" ]; then
  # Fall back to repo info for default branch name
  DEFAULT_BRANCH=$(gh api repos/$OWNER/$REPO --jq '.default_branch')
  SHA=$(gh api repos/$OWNER/$REPO/git/refs/heads/$DEFAULT_BRANCH \
    --jq '.object.sha')
fi
```

## Step 4 — Get the full recursive file tree

```bash
gh api "repos/$OWNER/$REPO/git/trees/$SHA?recursive=1" \
  --jq '.tree[].path' > "$SESSION/all_files.txt"

echo "Total files in repo: $(wc -l < "$SESSION/all_files.txt")"
```

## Step 5 — Filter for high-value integration files

These file patterns are most useful for understanding a downstream service's integration surface:

```bash
grep -iE \
  '(README\.md$|\.proto$|openapi\.(yaml|yml|json)$|swagger\.(yaml|yml|json)$|asyncapi\.(yaml|yml|json)$|graphql\.schema$|\.graphql$|\.env\.example$|\.env\.sample$|\.env\.template$|config\.example\.|example\.config\.|/api/|/apis/|/handler/|/handlers/|/endpoint/|/endpoints/|/route/|/routes/|client\.(go|ts|js|py|java|rb)$|/client/|/clients/|Client\.(go|ts|js|py|java|rb)$|/model/|/models/|/dto/|/dtos/|/schema/|/schemas/|/types/|Makefile$|docker-compose\.yml$)' \
  "$SESSION/all_files.txt" | head -50 > "$SESSION/relevant_files.txt"

echo "Relevant files identified: $(wc -l < "$SESSION/relevant_files.txt")"
```

### File type priority (read in this order)

| Priority | File types | Why |
|---|---|---|
| 1 (highest) | `INTEGRATION.md`, `DOWNSTREAM.md`, `INTEGRATION_GUIDE.md` | Explicit integration guide |
| 2 | `README.md` | Service overview |
| 3 | `*.proto`, `openapi.*`, `swagger.*`, `asyncapi.*`, `*.graphql` | API contract |
| 4 | `*client*`, `*Client*`, `/client/`, `/clients/` | Client SDK examples |
| 5 | `*.env.example`, `config.example.*` | Required configuration |
| 6 | `/model/`, `/dto/`, `/schema/`, `/types/` | Data structures |
| 7 | `Makefile`, `docker-compose.yml` | How to run the service |

## Step 6 — Download relevant files

```bash
while IFS= read -r filepath; do
  dir="$SESSION/$(dirname "$filepath")"
  mkdir -p "$dir"
  content=$(gh api "repos/$OWNER/$REPO/contents/$filepath" \
    --jq '.content' 2>/dev/null)
  if [ -n "$content" ]; then
    echo "$content" | base64 -d > "$SESSION/$filepath"
    echo "Downloaded: $filepath"
  fi
done < "$SESSION/relevant_files.txt"
```

## Step 7 — Full repo clone (fallback)

Use this only if:
- Fewer than 5 relevant files were found via filtering
- The user explicitly requests the full repo
- The relevant files exceed 50 (tree may be truncated)

```bash
gh repo clone $OWNER/$REPO "$SESSION/repo" -- --depth=1 --quiet
echo "Full repo cloned to $SESSION/repo"
```

After cloning, apply the same file pattern filtering within `$SESSION/repo/`.

## Step 8 — Verify what was downloaded

```bash
echo "=== Downloaded files ==="
find "$SESSION" -type f | grep -v 'all_files.txt\|relevant_files.txt' | sort

echo "=== Session size ==="
du -sh "$SESSION"
```

## Step 9 — Cleanup (after context is in Copilot's working memory)

```bash
rm -rf "$SESSION"
echo "Session files cleaned up: $SESSION"
```

**Important:** Run cleanup AFTER the agent has read all files into its context. The files are only needed for initial loading — once Copilot has read them, the local copies serve no purpose.

## Error handling

| Error | Action |
|---|---|
| `CONTEXT_FILE` not found | Ask user to confirm path; list root-level `.md` files as alternatives |
| Repo not accessible | Check `gh auth status`; ensure user has access to the repo |
| Tree too large (>100k nodes) | Fall back to targeted path fetching for known patterns |
| File download fails (binary/large) | Skip and note it; don't fail the whole session |
| `uuidgen` not available | Use `date +%s%N \| md5sum \| cut -c1-8` as fallback |
