# GitWhy Workflows

## When to Save Context

Save with GitWhy after any work where the reasoning matters for future developers or reviewers.

| Trigger | Why It Matters |
|---------|---------------|
| After significant implementation work | Preserves approach, decisions, and trade-offs |
| Before opening a PR | Gives reviewers the "why" behind the diff |
| After a debugging session | Documents root cause analysis and fix rationale |
| After an architecture decision | Records alternatives considered and reasons for choice |
| After a complex refactor | Explains what changed and why the new structure is better |
| When switching agents or context | Captures knowledge before it's lost |

## How to Save

### Via MCP

Generate context in `<context>` XML tags following the template in SKILL.md, then call:

```json
gitwhy_save({
  "markdown": "<context>\n  <title>JWT Authentication Setup</title>\n  <story>Phase 1 — Setup:\nUser asked me to add JWT auth...</story>\n  <reasoning>JWT over sessions for stateless API.\n    <decisions>\n      - RS256 for key rotation\n    </decisions>\n  </reasoning>\n  <files>\nsrc/auth/middleware.ts — new — JWT validation\n  </files>\n  <agent>claude-code (claude-opus-4)</agent>\n  <tags>auth, jwt</tags>\n</context>",
  "domain": "authentication",
  "topic": "jwt-implementation"
})
```

- `domain` and `topic` are optional (also accepted inside XML tags). Auto-categorized if omitted.
- Returns `{ "context_id": "ctx_a1b2c3d4", "message": "Context saved successfully" }`.
- Context ID, repository, branch, date, and commits are auto-populated by the CLI.

### Via CLI

Write the `<context>` XML content to a temporary file, then save:

```bash
git why save --file /tmp/gitwhy-context.md
```

Or pipe structured content directly:

```bash
echo '<context>
  <title>JWT Authentication Setup</title>
  <story>...</story>
  <reasoning>...</reasoning>
  <files>src/auth/middleware.ts — new — JWT validation</files>
</context>' | git why save
```

Optional flags:
- `--domain authentication` — specify domain
- `--topic jwt-implementation` — specify topic

### MCP Fallback to CLI

If the MCP tool fails or returns an error:
1. **Inform the user**: "The GitWhy MCP tool encountered an error. Falling back to CLI."
2. **Write the context content** to a temporary file (e.g., `/tmp/gitwhy-context.md`)
3. **Run**: `git why save --file /tmp/gitwhy-context.md`
4. The CLI output will show the saved context ID

This fallback works for ALL GitWhy operations — see the mapping table in SKILL.md.

## How to Retrieve

### By ID

**MCP:**
```json
gitwhy_get({ "id": "ctx_a1b2c3d4" })
```

**CLI:**
```bash
git why get ctx_a1b2c3d4
```

Returns the full structured markdown plus a dashboard URL.

### By Search

**MCP:**
```json
gitwhy_search({ "query": "authentication" })
```

**CLI:**
```bash
git why search "authentication"
```

Returns matching contexts with IDs, titles, domains, and dates.

### Browse the Tree

**MCP:**
```json
gitwhy_list()
```

**CLI:**
```bash
git why tree
```

Displays the hierarchical domain/topic structure:

```
.gitwhy/contexts/
  authentication/
    jwt-implementation/
      ctx_a1b2c3d4 — JWT Authentication Setup (2026-02-22)
    oauth-flow/
      ctx_e5f6g7h8 — OAuth Flow Configuration (2026-02-21)
  database/
    migration-strategy/
      ctx_m3n4o5p6 — Database Migration Plan (2026-02-20)
```

### List All Contexts

```bash
git why log
```

Shows one line per context: ID, date, agent, title, domain/topic.

## How to Push to PR

**MCP (two steps):**
```json
gitwhy_publish({ "ids": "[\"ctx_a1b2c3d4\"]" })
gitwhy_post_pr({ "ids": "[\"ctx_a1b2c3d4\"]" })
```

**CLI:**
```bash
git why push <context-id> --share
git why post-pr ctx_a1b2c3d4
```

The push command:
1. Syncs all unpushed contexts to the hosted API
2. With `--share`: promotes contexts from private to shared visibility
3. The `post-pr` command triggers PR comments on open PRs containing linked commits

## Example Prompts

Developers can tell their agent any of these:

| Prompt | What Happens |
|--------|-------------|
| "Save this session with GitWhy" | Agent generates structured context and calls `gitwhy_save` (MCP) or `git why save` (CLI) |
| "What context do we have on authentication?" | Agent calls `gitwhy_search(query="authentication")` or `git why search "auth"` |
| "Show me the context tree" | Agent calls `gitwhy_list()` or `git why tree` |
| "What contexts exist in the database domain?" | Agent calls `gitwhy_list(domain="database")` or `git why tree` |
| "Get the context for ctx_a1b2c3d4" | Agent calls `gitwhy_get(id="ctx_a1b2c3d4")` or `git why get ctx_a1b2c3d4` |
| "Are there any unsaved commits?" | Agent calls `gitwhy_status` or `git why log` |
| "Sync my contexts to the cloud" | Agent calls `gitwhy_sync` or `git why push <context-id>` |
| "Share my contexts with the team" | Agent calls `gitwhy_publish(ids=...)` or `git why push <context-id> --share` |
| "Post the context to the PR" | Agent calls `gitwhy_post_pr` or `git why post-pr` |

## Context Organization

Contexts are stored in a domain/topic tree under `.gitwhy/contexts/`:

- **Domains** — Top-level groupings (e.g., `authentication`, `database`, `api-design`)
- **Topics** — Specific subjects within a domain (e.g., `jwt-implementation`, `migration-strategy`)
- **Contexts** — Individual saved context files (`ctx_<id>.md`)

If you specify `domain` and `topic` when saving, those are used directly. Otherwise, GitWhy auto-categorizes based on the title and files changed. Unmatched contexts go to `uncategorized/general/`.
