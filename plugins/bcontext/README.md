# Bcontext plugin for Claude Code

Teaches Claude Code to work a Bcontext workspace natively: registers the MCP
server, loads the conventions skill and adds `/bcontext:brief`.

## Install

```
/plugin marketplace add Bcontext-dev/bcontext-plugin
/plugin install bcontext@bcontext
```

Then set the connection variables before launching Claude Code:

```bash
export BCONTEXT_TOKEN="bctx_live_…"             # agent token (Account or Connect tools → Agent token)
export BCONTEXT_URL="https://app.bcontext.dev"   # or your self-hosted origin
export BCONTEXT_WORKSPACE="my-workspace"         # only for user-scoped tokens
```

## What's inside

- `.mcp.json` — the `bcontext` MCP server (streamable HTTP at
  `$BCONTEXT_URL/mcp`, bearer from `$BCONTEXT_TOKEN`, workspace header from
  `$BCONTEXT_WORKSPACE`).
- `skills/bcontext/SKILL.md` — the behaviour pack for the 28-tool surface:
  `brief` first, H2 blocks and `/n/<id>#<block>` refs, subtasks inside a
  task, goals and key results, datasets, reusable tags, `blocked_by` /
  `contributes_to` edges, the `if_updated_at` rule and the 409 rebase loop,
  scoped retrieval with `ask_nexo` / `ask_rag`, credentials you use without
  reading, and `list_changes` re-sync.
- `commands/brief.md` — `/bcontext:brief [since]`: shipped, decisions,
  blockers and newly unblocked work, every item with its `/n/<id>` ref.

Bcontext stores typed nodes (docs, tasks, decisions, meetings, goals,
datasets, credentials…). Reusable tags classify them across views and the
graph; an optional `parent_id` expresses a real semantic hierarchy.
