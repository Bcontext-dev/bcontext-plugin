---
name: bcontext
description: How to work a Bcontext workspace correctly — CLI-first surface choice, H2 block addressing, optimistic concurrency, typed dependencies, cited retrieval, and cheap re-sync. Use whenever reading from or writing to a Bcontext workspace (bcontext.es or self-hosted), via the bcontext/bx CLI or the mcp__bcontext__* tools.
---

# Working with Bcontext

Bcontext is a multi-writer knowledge graph: humans and other agents edit the same workspace while you work. Every convention below exists to keep concurrent sessions from destroying each other's work and to keep every claim verifiable.

## Pick the right surface

**Shell available (Claude Code, CI)?** Prefer the CLI — `bcontext`, alias `bx` (`npx @bcontext/cli`). MCP injects its tool schema into context every turn; a shell command costs ~0 standing tokens. Config: `BCONTEXT_TOKEN`, `BCONTEXT_URL`, `BCONTEXT_WORKSPACE` env vars, or `bx login`.

**No shell (hosted client)?** Use the `mcp__bcontext__*` tools — identical contracts.

Anything without a CLI subcommand: `bx mcp <tool> '<json-args>'` (also `bx mcp tools/list` to discover).

## Orient: ask first, browse second

1. `bx ask "<question>"` (MCP: `ask_rag`) — synthesized answer where **every claim carries an inline [N] citation**; sources print as `/n/<node_id>#<block_id>` refs. Follow a ref with `bx get` to verify before acting on it. It refuses to answer beyond the corpus — trust that.
2. `bx search "<query>"` — one hit per node, matched H2 blocks nested, each with a paste-ready `ref`.
3. `bx get <id>` — full envelope: content, `inbound_links` (who references this — discovery you can't search for), `outbound_links`, `dependencies` (blocker status), `attachments`.
4. `bx unblocked` — the "what can I start now" query.

## Write safely (multi-writer rules)

- **Guard every content edit**: read the node, hold its `updated_at`, pass it back — `bx update <id> --md - --if-updated-at <ts>` (MCP: `update_node({ id, if_updated_at, patch })`). A **409** means another session wrote first: re-fetch, rebase your change on top, retry with the new timestamp. Never retry a 409 blindly and never omit the guard on content_md.
- **Never rewrite a body to flip a checkbox**: `bx toggle <node-id> <index> [--block <slug>]` flips one `- [ ]` atomically. A bad index returns the item inventory — self-correct from it.
- **Structure with H2**: every `##` heading becomes a stable addressable block (`/n/<id>#<slug-of-heading>`). Write H2s deliberately; cite blocks, not whole nodes. Creating a `decision`/`adr`/`meeting`/`bug` with no body pre-fills its standard skeleton — fill it, don't fight it.
- **MCP shape gotcha**: `create_node` takes fields flat; `update_node` nests them inside `patch`.

## Dependencies are edges, not prose

Never write "depends on X" in markdown. Instead:

```
bx link <task-id> <blocker-id> --relation blocked_by    # task waits on blocker
```

Relations: `blocked_by`, `references`, `caused_by`, `relates_to`. Cycles are rejected (you'll get the offending path); `blocked_by` to something that can never be done (folder/decision) is rejected — use `references`. `bx get` on a task then reports live blocker status, and completing a blocker instantly surfaces dependents in `bx unblocked`.

## Long sessions: re-sync, don't re-read

`bx changes --since <ts-of-your-last-sync>` returns everything that changed (create/update/rename/move/delete, with actor kind). Pass the newest `ts` back as the next `--since`. Filter one node's history with `--node <id>` (MCP: `list_changes({ node_id })`).

## Misc contracts worth knowing

- New content is searchable after ~15s (async embedding) — `get`/`tree` see it immediately.
- Deletes cascade and echo their blast radius (`deleted_count`/`deleted_ids`); attachments die with their node.
- `bx attach <node-id> <file.png>` for screenshots (png/jpeg/webp/gif ≤ 4 MB); embed the returned URL as markdown.
- Errors are self-correcting by design: read them — they name the offending key, list valid options, or include the recovery recipe.
