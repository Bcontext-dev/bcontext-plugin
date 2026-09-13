# Bcontext — Claude Code plugin

The Claude Code integration for [Bcontext](https://bcontext.dev), the
AI-native context layer where people and agents share one workspace. This
repository is a plugin marketplace with a single plugin, `bcontext`, which:

- registers the Bcontext **MCP server** (`/mcp`, streamable HTTP, bearer
  token) so the `mcp__bcontext__*` tools are available in every session —
  or, per project, you connect one workspace's endpoint with OAuth (below);
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

Then connect, one of two ways.

**Per project (recommended)** — the workspace's own MCP endpoint, OAuth in
the browser, no secret, a `.mcp.json` you commit with the project:

```bash
claude mcp add -s project --transport http bcontext-<org>-<ws> https://app.bcontext.dev/mcp/<org>/<ws>
```

The token Claude Code obtains is bound to that workspace: sessions in that
folder can only read and write there.

**Per user, with an agent token** — the plugin's `.mcp.json` reads these
before launch:

```bash
export BCONTEXT_TOKEN="bctx_live_…"             # agent token (Account or Connect tools → Agent token)
export BCONTEXT_URL="https://app.bcontext.dev"   # or your self-hosted origin
export BCONTEXT_WORKSPACE="my-workspace"         # only for user-scoped tokens
```

A workspace-scoped token is bound to one workspace; a user token picks one
per call with `BCONTEXT_WORKSPACE`. `BCONTEXT_URL` is the origin (the plugin
appends `/mcp`); to bind a token flow to `/mcp/<org>/<ws>` use `claude mcp
add` with an `Authorization` header instead of the plugin's entry.

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
`bcontext` (`src/lib/mcp/server.ts`: `TOOLS` are the native ones,
`CAPABILITY_ROUTING_TOOLS` the two that reach external tools and skills),
update `SKILL.md`, the tool count in `plugin.json` and bump the version. Branches: `feat/*` → PR to `dev` →
promote to `main`; there is no build step.
