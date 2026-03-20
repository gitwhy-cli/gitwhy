# GitWhy Agent Skills

Agent skills for [GitWhy](https://gitwhy.dev) — save the reasoning, decisions, and trade-offs behind AI-generated code.

## Install

```bash
npx skills add gitwhy-cli/gitwhy
```

Or install for a specific agent:

```bash
npx skills add gitwhy-cli/gitwhy --agent claude
npx skills add gitwhy-cli/gitwhy --agent cursor
npx skills add gitwhy-cli/gitwhy --agent codex
```

## What's Included

### `gitwhy` skill

Teaches your AI coding agent to save, retrieve, and search structured context linked to git commits. Works via MCP tools or CLI commands.

**Trigger phrases:**
- "Save this session with GitWhy"
- "What context do we have on authentication?"
- "Show me the context tree"
- "Post the context to the PR"

## Prerequisites

Install the GitWhy CLI first:

```bash
curl -fsSL https://gitwhy.dev/install.sh | sh
```

Or via Homebrew:

```bash
brew install gitwhy-cli/tap/git-why
```

Then run setup:

```bash
git why setup
```

## Links

- [GitWhy](https://gitwhy.dev)
- [Documentation](https://docs.gitwhy.dev)
- [Dashboard](https://app.gitwhy.dev)
