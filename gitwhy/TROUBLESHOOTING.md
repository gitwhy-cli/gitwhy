# GitWhy Troubleshooting

## MCP Tool Failures — CLI Fallback

If any MCP tool returns an error or is unavailable, **fall back to the CLI equivalent**:

| MCP Tool | CLI Fallback |
|----------|-------------|
| `gitwhy_save` | Write content to file, then `git why save --file context.md` |
| `gitwhy_get` | `git why get <id>` |
| `gitwhy_search` | `git why search "<query>"` |
| `gitwhy_list` | `git why tree` or `git why log` |
| `gitwhy_status` | `git why log` (list contexts) |
| `gitwhy_sync` | `git why push <context-id>` |
| `gitwhy_publish` | `git why push <context-id> --share` |
| `gitwhy_post_pr` | `git why post-pr [ids...]` |

**Important:** Always inform the user when falling back to CLI: "The GitWhy MCP tool encountered an error. Falling back to CLI."

## Common Errors

### Validation Failure: Missing Required Sections

**Error:** `validation error: missing required section: <section-name>`

When using XML format (`<context>` tags), the following tags are **required**:

| Required Tag | What to Include |
|-------------|----------------|
| `<title>` | Short title of what was done |
| `<story>` | Phase-organized engineering journal |
| `<reasoning>` | Why this approach was chosen |
| `<files>` | Files changed with status |

**Fix:** Ensure your context includes all four required tags before calling `gitwhy_save` or `git why save`.

### No API Key Configured

**Error:** `no API key found — run 'git why setup' to set up`

This occurs when `git why push <context-id>` or cloud operations cannot find credentials.

**Fix:**
1. Run `git why setup`
2. Sign up at `https://app.gitwhy.dev/signup` when prompted
3. Paste the API key when asked — it is stored at `~/.gitwhy/credentials`

Note: Local commands (`save`, `get`, `log`, `tree`, `search`) work fully offline without an API key. Only `push`, `sync`, `publish`, and `post-pr` require it.

### Context Not Found

**Error:** `context not found: ctx_xxxxxxxx`

**Fix:**
- Verify the ID format: `ctx_` followed by exactly 8 alphanumeric characters (e.g., `ctx_a1b2c3d4`)
- Run `git why log` to see all saved context IDs in this repository
- Ensure you're in the correct repository — contexts are repo-local
- Check the domain/topic tree with `git why tree` to browse by structure

### Wrong Repo Root (Home Directory)

**Error:** `repo_root is your home directory`

The MCP server sometimes detects the home directory as the repo root.

**Fix (MCP):** Pass `repo_root` with your project's absolute path:
```json
gitwhy_status({ "repo_root": "/Users/dev/myproject" })
gitwhy_save({ "markdown": "...", "repo_root": "/Users/dev/myproject" })
```

**Fix (CLI):** Run the command from your project directory — the CLI auto-detects the git root.

### Push Failed

**Error:** `push failed: <network error>`

**Fix:**
- Check your internet connection
- Verify API key is valid: `git why setup` (re-enter key if needed)
- Push is idempotent — safe to retry: `git why push <context-id>`
- Local contexts are never lost — they remain in `.git/gitwhy/contexts/`

## Diagnostic Commands

| Command | What It Shows |
|---------|--------------|
| `git why log` | All saved contexts with IDs, dates, titles, agents, domains |
| `git why tree` | Domain/topic tree structure with context files |
| `git why search <query>` | Search across context titles, prompts, reasoning, files |
| `git why get <id>` | Full content of a specific context |
| `git why --version` | Installed GitWhy version |

## How to Verify Installation

### 1. Binary Installed

```bash
git why --version
# Expected: git-why version X.Y.Z
```

### 2. MCP Config Registered (if MCP method selected)

Check your agent's MCP configuration for:

```json
{
  "mcpServers": {
    "gitwhy": {
      "command": "git-why",
      "args": ["mcp"]
    }
  }
}
```

### 3. Local Storage Directory Exists

```bash
ls .git/gitwhy/
# Expected: contexts/ directory
```

If any check fails, re-run `git why setup` to repair the setup.
