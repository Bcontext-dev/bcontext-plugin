# Bcontext plugin for Claude Code

Teaches Claude Code to work a Bcontext workspace natively: registers the MCP server, loads the conventions skill (H2 blocks, optimistic concurrency, typed dependencies, cited retrieval, CLI-first surface choice), and adds `/bcontext:brief`.

## Install

```
/plugin marketplace add Bcontext-dev/bcontext-plugin
/plugin install bcontext@bcontext
```

Then set the connection env vars before launching Claude Code:

```
export BCONTEXT_TOKEN="bctx_live_…"      # workspace token (Settings → Tokens)
export BCONTEXT_URL="https://bcontext.es"  # or your self-hosted origin
export BCONTEXT_WORKSPACE="my-workspace"   # only needed for user-scoped tokens
```

## What's inside

- `.mcp.json` — the `bcontext` MCP server (streamable HTTP, bearer from `$BCONTEXT_TOKEN`).
- `skills/bcontext/` — the behavior pack: surface choice, scoped RAG, reusable tags, semantic hierarchy, the 409 rebase loop, block refs, `blocked_by` edges, and `list_changes` re-sync.
- `commands/brief.md` — `/bcontext:brief [since]`: standup brief with `/n/<id>` refs.

The CLI (`npx @bcontext/cli`, binaries `bcontext` and `bx`) is recommended alongside: same token, ~0 standing context tokens per call. See `/docs/cli`.

Bcontext stores real typed nodes. Reusable tags classify them across views and
the graph; optional `parent_id` links express semantic relationships such as
epic/subtask or document/part.
