---
name: gitwhy-context-saving
description: Saves, retrieves, and searches structured context (reasoning, decisions, trade-offs) behind AI-generated code, linked to git commits. Use when the developer says "save this session with GitWhy", asks about past context or decisions, or wants to push context to a PR. Also use after significant implementation work, debugging sessions, or architecture decisions.
---

# GitWhy — Context Layer for Git

Save, retrieve, and share the reasoning behind AI-generated code — tied to commits.

**Workflows**: See [WORKFLOWS.md](WORKFLOWS.md) for when/how to save, retrieve, and push.
**Troubleshooting**: See [TROUBLESHOOTING.md](TROUBLESHOOTING.md) for common errors and diagnostics.

## Two Ways to Use GitWhy

GitWhy works via **MCP tools** (if available) or **CLI commands** (always available). Both are equivalent — use whichever works in your environment.

| MCP Tool | CLI Command | Purpose |
|----------|------------|---------|
| `gitwhy_status` | `git why log` | Check saved contexts and pending commits |
| `gitwhy_save` | `git why save --file context.md` | Save structured context |
| `gitwhy_get` | `git why get <id>` | Retrieve context by ID |
| `gitwhy_search` | `git why search "<query>"` | Search contexts by keyword |
| `gitwhy_list` | `git why tree` / `git why log` | Browse domain/topic structure |
| `gitwhy_sync` | `git why push <context-id>` | Upload to cloud (private) |
| `gitwhy_publish` | `git why push <context-id> --share` | Share with team |
| `gitwhy_post_pr` | `git why post-pr [context-id...]` | Post to GitHub PR |

**MCP fallback rule:** If an MCP tool fails or is unavailable, inform the user and use the equivalent CLI command instead. Every MCP operation has a CLI equivalent.

## Context Format (XML Input)

When saving context, wrap content in `<context>` XML tags. The CLI renders this into rich markdown for storage.

```xml
<context>
  <!-- Required -->
  <title>Short title of what was done</title>
  <story>
    Organize by phases. Write in first-person engineering journal style.

    Phase 1 — Setup:
    What user asked, what you did, challenges faced, how you resolved them.
    Include back-and-forth with the user where it shaped the outcome.

    Phase 2 — Implementation:
    Technical details, decisions made during coding, problems solved.
  </story>
  <reasoning>
    Why you chose this approach.

    <decisions>
      - RS256 over HS256 — allows key rotation without redeploying
      - 24h token expiry — balances security vs UX
    </decisions>

    <rejected>
      - Session cookies — requires server-side session store
      - OAuth2 external provider — overkill for internal service
    </rejected>

    <tradeoffs>
      - No refresh tokens in v1 — simplifies MVP but means 24h hard limit
    </tradeoffs>
  </reasoning>
  <files>
    src/auth/middleware.ts — new — Token verification middleware
    src/routes/login.ts — modified — Added token signing on login
    package.json — modified — Added jsonwebtoken dependency
  </files>

  <!-- Optional -->
  <agent>claude-code (claude-opus-4)</agent>
  <tags>auth, jwt, middleware</tags>
  <tools>MCP: nia, sequential-thinking</tools>
  <verification>All 14 tests passing. Build successful.</verification>
  <risks>No rate limiting on login endpoint yet.</risks>
</context>
```

### Tag Requirements

| Tag | Required | Notes |
|-----|----------|-------|
| `<title>` | **Yes** | Short title of the work done |
| `<story>` | **Yes** | Phase-organized engineering journal. First-person, chronological. |
| `<reasoning>` | **Yes** | Why this approach. Nest `<decisions>`, `<rejected>`, `<tradeoffs>` inside. |
| `<files>` | **Yes** | One per line. Flexible format: `path — status — desc` or just `path` |
| `<agent>` | Optional | Agent name and model |
| `<tags>` | Optional | Comma-separated keywords for discovery |
| `<tools>` | Optional | MCPs, CLI tools, resources used |
| `<verification>` | Optional | Test results, build status |
| `<risks>` | Optional | Open questions, follow-up items |

Context ID, repository, branch, date, domain/topic, and commits are auto-populated by the CLI.

## Quick Start

1. Check status: `gitwhy_status` (MCP) or `git why log` (CLI)
2. Developer says: **"Save this session with GitWhy"**
3. You generate context in `<context>` XML tags using the template above
4. Save context:
   - **MCP:** Call `gitwhy_save(markdown="<context>...</context>")`
   - **CLI:** Write content to file, then run `git why save --file context.md`
   - **CLI (pipe):** `echo '<context>...</context>' | git why save`
5. GitWhy parses XML, renders rich markdown, generates context ID, stores locally
6. Context ID is returned — use it for retrieval later

To retrieve past context:
- By ID: `gitwhy_get(id="ctx_a1b2c3d4")` or `git why get ctx_a1b2c3d4`
- By search: `gitwhy_search(query="authentication")` or `git why search "auth"`
- Browse tree: `gitwhy_list()` or `git why tree`

To share with team:
- Sync to cloud: `gitwhy_sync` or `git why push <context-id>`
- Publish: `gitwhy_publish(ids=...)` or `git why push <context-id> --share`
- Post to PR: `gitwhy_post_pr` or `git why post-pr [context-id...]`
