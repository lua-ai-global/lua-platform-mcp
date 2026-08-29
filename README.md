# lua-platform-mcp

Read-only MCP server for the [Lua AI platform](https://heylua.ai) — exposes 5 tools that let any MCP-aware client (Cline, Cursor, Claude Code, Codex CLI, Continue, Zed, etc.) query Lua agent state mid-conversation.

## Tools exposed

| Tool | What it returns |
|---|---|
| `list_agents` | All agents accessible to the authenticated user (`{id, name, orgId, orgName}[]`) |
| `get_agent` | Public-facing config for one agent by ID |
| `list_primitive_versions` | Versions of a skill / webhook / job / preprocessor / postprocessor / persona |
| `get_deployment_status` | What's live in production right now (composed from per-type version endpoints) |
| `tail_logs` | Structured log fetch (mirrors `lua logs --json`) |

All tools are **read-only** — this server cannot push, deploy, or modify any state.

## Install

### Cline / any MCP client (recommended)

After cloning + building, register the server in your client's MCP config. For Cline, that's the `cline_mcp_settings.json`:

```json
{
  "mcpServers": {
    "lua-platform": {
      "command": "node",
      "args": ["/absolute/path/to/lua-platform-mcp/dist/server.js"]
    }
  }
}
```

### From source

```bash
git clone https://github.com/lua-ai-global/lua-platform-mcp
cd lua-platform-mcp
npm install
npm run build
# Now dist/server.js is the entry point
```

### From npm (once published)

```bash
npx lua-platform-mcp   # one-shot
# or, in MCP config:
# "command": "npx", "args": ["-y", "lua-platform-mcp"]
```

## Authentication

The server reads your Lua API key in this order:

1. `LUA_API_KEY` environment variable
2. `~/.lua-cli/credentials` (written by `lua auth configure`)
3. `.env` file in CWD

Credentials are opaque to this MCP server: it selects the first available source and forwards the value unchanged. Never paste a credential into an MCP request or a model conversation. Configure it locally with the Lua CLI, your shell environment, or a local `.env` file instead.

For the copy-paste-safe local setup, use the Lua CLI before starting your MCP client:

```bash
npm install -g lua-cli
lua auth configure
```

That writes `~/.lua-cli/credentials`, which the server reads automatically. Alternatively, create a credential in [admin.heylua.ai](https://admin.heylua.ai) and set its real value in the MCP host's local environment.

## Verify it works

After registering, restart your MCP client and ask: *"What lua-platform tools do you have?"*

You should see all 5 tools listed under the `mcp__lua-platform__*` prefix (Cline / Cursor / Claude Code) or as direct names (Codex / Continue).

Try: *"List my Lua agents."*  → should return your agents, IDs, and orgs.

## Architecture

- `src/server.ts` — MCP stdio bootstrap (uses `@modelcontextprotocol/sdk`)
- `src/tools/*.mjs` — one file per tool, each exporting `{ name, description, inputSchema, handler }`
- `src/auth.mjs` — credential resolution (mirrors lua-cli's chain)
- `src/api-client.mjs` — typed HTTP wrapper around `https://api.heylua.ai`
- `dist/server.js` — esbuild-bundled single-file output (built via `npm run build`)

## Why "read-only"?

Mutating operations (`lua push`, `lua deploy`, etc.) live in the Lua CLI directly, behind the §3.3 deploy-safety contract that requires explicit user confirmation. We deliberately don't expose them via MCP because MCP tool calls don't carry the human-confirmation step. Read tools are safe; write tools belong in the CLI.

If you want a richer "build agents inside your IDE" experience (slash commands, scaffolding, deploy gates), install one of the IDE-specific Lua plugins instead — they bundle this MCP server alongside the workflow:

- [lua-ai-global/cursor-lua-agent-builder](https://github.com/lua-ai-global/cursor-lua-agent-builder) (Cursor)
- [lua-ai-global/claude-code-lua-plugin](https://github.com/lua-ai-global/claude-code-lua-plugin) (Anthropic Claude Code)
- (Codex CLI port coming soon)

## License

[MIT](./LICENSE) © Lua AI
