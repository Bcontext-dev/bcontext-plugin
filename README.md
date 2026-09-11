# Bcontext — Claude Code plugin

The Claude Code integration for [Bcontext](https://bcontext.dev), the
AI-native context layer where people and agents share one workspace. This
repository is a plugin marketplace with a single plugin, `bcontext`, which:

- registers the Bcontext **MCP server** (`/mcp`, streamable HTTP, bearer
  token) so the `mcp__bcontext__*` tools are available in every session;
- loads the **conventions skill** — how an agent works a workspace well:
  start with `brief`, H2 blocks as addressable refs, subtasks inside a task,
  goals, typed dependencies, cited retrieval, the `if_updated_at` rule and
  what to write down for the next reader;
- adds **`/bcontext:brief`**, a standup brief of the active workspace.

The product itself (API, MCP server, permission engine) lives in
[`bcontext`](https://github.com/Bcontext-dev/bcontext); the web app in
[`bcontext-web`](https://github.com/Bcontext-dev/bcontext-web); the
marketing site in [`landing`](https://github.com/Bcontext-dev/landing).

## Install

```
/plugin marketplace add Bcontext-dev/bcontext-plugin
/plugin install bcontext@bcontext
```

Set the connection variables before launching Claude Code:

```bash
export BCONTEXT_TOKEN="bctx_live_…"             # agent token (Account or Connect tools → Agent token)
export BCONTEXT_URL="https://app.bcontext.dev"   # or your self-hosted origin
export BCONTEXT_WORKSPACE="my-workspace"         # only for user-scoped tokens
```

A workspace-scoped token is bound to one workspace; a user token picks one
per call with `BCONTEXT_WORKSPACE`.

## Layout

```
.claude-plugin/marketplace.json    the marketplace (one plugin)
plugins/bcontext/
  .claude-plugin/plugin.json       name, version, description
  .mcp.json                        the `bcontext` MCP server entry
  skills/bcontext/SKILL.md         the conventions skill
  commands/brief.md                /bcontext:brief
```

See [`plugins/bcontext/README.md`](plugins/bcontext/README.md) for the
plugin's own notes.

## Keeping it in sync

The skill names concrete MCP tools. When tools or contracts change in
`bcontext` (`src/lib/mcp/server.ts`), update `SKILL.md`, the tool count in
`plugin.json` and bump the version. Branches: `feat/*` → PR to `dev` →
promote to `main`; there is no build step.
