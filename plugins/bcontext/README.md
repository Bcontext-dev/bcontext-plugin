# Bcontext plugin for Claude Code

Teaches Claude Code to work a Bcontext workspace natively: registers the MCP
server, loads the conventions skill and adds `/bcontext:brief`.

## Install

```
/plugin marketplace add Bcontext-dev/bcontext-plugin
/plugin install bcontext@bcontext
```

Then connect. Two ways, depending on whether the connection belongs to a
project or to you:

**Per project (recommended)** — one workspace, OAuth, no secret to manage,
shareable through git:

```bash
claude mcp add -s project --transport http bcontext-<org>-<ws> https://app.bcontext.dev/mcp/<org>/<ws>
```

This writes a `.mcp.json` in the project folder (commit it: it holds only
the URL) and Claude Code signs you in through the browser on first use. The
token it obtains is bound to that workspace, so every session in that folder
writes to the right place and cannot write anywhere else.

**Per user, with an agent token** — the plugin's own `.mcp.json`, driven by
environment variables set before launching Claude Code:

```bash
export BCONTEXT_TOKEN="bctx_live_…"             # agent token (Account or Connect tools → Agent token)
export BCONTEXT_URL="https://app.bcontext.dev"   # or your self-hosted origin
export BCONTEXT_WORKSPACE="my-workspace"         # only for user-scoped tokens
```

`BCONTEXT_URL` is the origin; the plugin appends `/mcp`. To point a token
flow at one workspace's endpoint (`/mcp/<org>/<ws>`) use `claude mcp add` as
above with `--header "Authorization: Bearer $BCONTEXT_TOKEN"` — the plugin's
template stays on the origin so the default keeps working for user tokens
that switch workspace per call.

## What's inside

- `.mcp.json` — the `bcontext` MCP server (streamable HTTP at
  `$BCONTEXT_URL/mcp`, bearer from `$BCONTEXT_TOKEN`, workspace header from
  `$BCONTEXT_WORKSPACE`).
- `skills/bcontext/SKILL.md` — the behaviour pack for the 28 native tools
  and the two routing tools (`capability_search` / `capability_call`, how
  external tools and skills are found and run): `brief` first, H2 blocks
  and `/n/<id>#<block>` refs, subtasks inside a task, goals and key results,
  datasets, reusable tags, `blocked_by` / `contributes_to` edges, the
  `if_updated_at` rule and the 409 rebase loop, scoped retrieval with
  `ask_nexo` / `ask_rag`, credentials you use without reading, and
  `list_changes` re-sync.
- `commands/brief.md` — `/bcontext:brief [since]`: shipped, decisions,
  blockers and newly unblocked work, every item with its `/n/<id>` ref.

Bcontext stores typed nodes (docs, tasks, decisions, meetings, goals,
datasets, credentials…). Reusable tags classify them across views and the
graph; an optional `parent_id` expresses a real semantic hierarchy.
