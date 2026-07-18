---
name: bcontext
description: How to work a Bcontext workspace correctly — CLI-first surface choice, H2 block addressing, optimistic concurrency, typed dependencies, cited retrieval, authorized external capabilities, and cheap re-sync. Use whenever reading from or writing to a Bcontext workspace (bcontext.es or self-hosted), via the bcontext/bx CLI or the mcp__bcontext__* tools.
---

# Working with Bcontext

Bcontext is a multi-writer knowledge graph: humans and other agents edit the same workspace while you work. Every convention below exists to keep concurrent sessions from destroying each other's work and to keep every claim verifiable.

## Pick the right surface

**Shell available (Claude Code, CI)?** Prefer the CLI — `bcontext`, alias `bx` (`npx @bcontext/cli`). MCP injects its tool schema into context every turn; a shell command costs ~0 standing tokens. Config: `BCONTEXT_TOKEN`, `BCONTEXT_URL`, `BCONTEXT_WORKSPACE` env vars, or `bx login`.

**No shell (hosted client)?** Use the `mcp__bcontext__*` tools — identical contracts.

Anything without a CLI subcommand: `bx mcp <tool> '<json-args>'` (also `bx mcp tools/list` to discover).

## External capabilities are live grants

Tools named `<provider>__<capability>__<tool>` execute in an external provider through Bcontext's gateway. `tools/list` is authoritative: it exposes only this principal's active grants, and a tool disappearing means its grant or capability was revoked. Bcontext re-authorizes every call, so never rely on a stale cached tool list.

Treat these as open-world actions. Inspect the registered description/schema before irreversible calls, keep arguments minimal, and never retry an external action blindly. Retry only when the returned error explicitly says it is safe; Bcontext supplies idempotency only for providers whose registered contract supports it.

## Orient: ask first, browse second

1. `bx ask "<question>"` (MCP: `ask_rag`) — synthesized answer where **every claim carries an inline [N] citation**; sources print as `/n/<node_id>#<block_id>` refs. Follow a ref with `bx get` to verify before acting on it. It refuses to answer beyond the corpus — trust that.
2. `bx search "<query>"` — one hit per node, matched H2 blocks nested, each with a paste-ready `ref`.
3. `bx get <id>` — full envelope: content, `inbound_links` (who references this — discovery you can't search for), `outbound_links`, `dependencies` (blocker status), `attachments`.
4. `bx unblocked` — the "what can I start now" query.

## Choose retrieval scope explicitly

`search_nodes` and `ask_rag` default to the complete workspace. Keep that default for broad orientation. When the question is intentionally local, pass the versioned `scope` contract (schema version 1): `view`, `tags` (`tags_any` / `tags_all`), `selection`, or `neighborhood`. CLI equivalents:

```
bx search "launch risk" --view <view-id>
bx ask "what changed?" --tags-any product,tech
bx search "impact" --select <id,id>
bx ask "nearby decisions" --near <node-id> --depth 1
```

Explicit scopes are strict by default: filtering happens before vector/keyword retrieval. `--broad` allows graph expansion outside the original set and every such neighbor is marked `outside_scope`. `--scope auto` (or `--auto`) accepts mixed hints, boosts their candidate lanes, and always retains a visible workspace/global lane; never describe auto as a permission boundary. Results report scope type, size, workspace size, lane boosts, and whether broad expansion occurred. Tags classify knowledge; they are never ACLs, never separate indexes, and tag edits do not require re-embedding.

## Write safely (multi-writer rules)

- **Guard every content edit**: read the node, hold its `updated_at`, pass it back — `bx update <id> --md - --if-updated-at <ts>` (MCP: `update_node({ id, if_updated_at, patch })`). A **409** means another session wrote first: re-fetch, rebase your change on top, retry with the new timestamp. Never retry a 409 blindly and never omit the guard on content_md.
- **Never rewrite a body to flip a checkbox**: `bx toggle <node-id> <index> [--block <slug>]` flips one `- [ ]` atomically. A bad index returns the item inventory — self-correct from it.
- **Structure with H2**: every `##` heading becomes a stable addressable block (`/n/<id>#<slug-of-heading>`). Write H2s deliberately; cite blocks, not whole nodes. Creating a `decision`/`adr`/`meeting`/`bug` with no body pre-fills its standard skeleton — fill it, don't fight it.
- **MCP shape gotcha**: `create_node` takes fields flat; `update_node` nests them inside `patch`.

## Classify with tags; use parent only for meaning

- Create only real knowledge nodes: `doc`, `task`, `decision`, `meeting`,
  `bug`, `adr`, `entity`, or `skill`. Reusable workspace tags provide
  aboutness and replace single-purpose containers. A node can carry several
  tags.
- Call `list_tags` before writes, then pass stable ids as `tag_ids` to
  `create_node` or `update_node`. Updating `tag_ids` replaces the complete
  assignment set. Tags never become nodes or embeddings and never grant
  permissions.
- `parent_id` is optional semantic hierarchy between real nodes only, such as
  epic/subtask or document/part. Do not use it for placement. Most nodes can
  stay at root and be discovered through tags, views, search, and graph links.

## Dependencies are edges, not prose

Never write "depends on X" in markdown. Instead:

```
bx link <task-id> <blocker-id> --relation blocked_by    # task waits on blocker
```

Relations: `blocked_by`, `references`, `caused_by`, `relates_to`. Cycles are rejected (you'll get the offending path); use `blocked_by` only when the target is actionable work that can reach done, and use `references` for knowledge such as decisions. `bx get` on a task then reports live blocker status, and completing a blocker instantly surfaces dependents in `bx unblocked`.

## Long sessions: re-sync, don't re-read

`bx changes --since <ts-of-your-last-sync>` returns everything that changed (create/update/rename/move/delete, with actor kind). Pass the newest `ts` back as the next `--since`. Filter one node's history with `--node <id>` (MCP: `list_changes({ node_id })`).

## Misc contracts worth knowing

- New content is searchable after ~15s (async embedding) — direct `get` and workspace reads see it immediately.
- Before deleting, inspect the exact node, semantic children, references, and attachments; deletion is irreversible.
- `bx attach <node-id> <file.png>` for screenshots (png/jpeg/webp/gif ≤ 4 MB); embed the returned URL as markdown.
- Errors are self-correcting by design: read them — they name the offending key, list valid options, or include the recovery recipe.
