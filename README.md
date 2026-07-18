# Bcontext — Claude Code plugin marketplace

Claude Code integration for [Bcontext](https://bcontext.es): registers the MCP server, installs the agent-conventions skill (reusable tags, semantic hierarchy, block refs, optimistic concurrency, typed dependencies, cited retrieval, CLI-first surface choice), and adds `/bcontext:brief`.

## Install

```
/plugin marketplace add Bcontext-dev/bcontext-plugin
/plugin install bcontext@bcontext
```

Set the connection env vars before launching Claude Code:

```bash
export BCONTEXT_TOKEN="bctx_live_…"        # workspace token (Settings → Tokens)
export BCONTEXT_URL="https://bcontext.es"  # or your self-hosted origin
export BCONTEXT_WORKSPACE="my-workspace"   # only needed for user-scoped tokens
```

See [`plugins/bcontext/README.md`](plugins/bcontext/README.md) for what's inside. Pairs well with the [`@bcontext/cli`](https://github.com/Bcontext-dev/bcontext-cli) (`bx`) for ~0-token workspace calls from the shell.
