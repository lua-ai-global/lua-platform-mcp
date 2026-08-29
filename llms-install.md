# LLM install instructions

This file gives Cline (or any AI agent installing this MCP server on a user's behalf) the exact steps to set it up successfully on first try.

## Prereqs you can assume

- Node ≥18
- npm

## Install steps

1. Clone the repo to a stable location:
   ```bash
   git clone https://github.com/lua-ai-global/lua-platform-mcp ~/.local/share/lua-platform-mcp
   ```

2. Install deps and build the bundle:
   ```bash
   cd ~/.local/share/lua-platform-mcp
   npm install
   npm run build
   ```
   This produces `dist/server.js` — a single-file esbuild bundle that's the MCP entry point.

3. Register the server in the MCP config file. For Cline that's `cline_mcp_settings.json` (location varies — typically `~/Library/Application Support/Cursor/User/globalStorage/saoudrizwan.claude-dev/settings/cline_mcp_settings.json` on macOS or analogous paths on other OSes). Add or merge:
   ```json
   {
     "mcpServers": {
       "lua-platform": {
         "command": "node",
         "args": ["/absolute/path/to/lua-platform-mcp/dist/server.js"],
         "env": {
           "LUA_API_KEY": "${env:LUA_API_KEY}"
         }
       }
     }
   }
   ```
   Use the absolute path — relative paths are unreliable across MCP client implementations.

4. Keep credentials on the user's machine. Do **not** ask the user to paste a secret into this conversation or any MCP request. Tell them to choose one local setup path:
   - Run `lua auth configure` after installing `lua-cli`; it writes `~/.lua-cli/credentials`.
   - Set `LUA_API_KEY` in the MCP host's local environment.
   - Add `LUA_API_KEY` to a local `.env` file in the MCP working directory.

   The server selects sources in that order and forwards either legacy or dotted typed credentials unchanged. If the user needs to create or manage a credential, direct them to https://admin.heylua.ai rather than asking for the secret.

5. Restart Cline. Verify by asking: *"What lua-platform MCP tools are available?"* — expect 5 tools: `list_agents`, `get_agent`, `list_primitive_versions`, `get_deployment_status`, `tail_logs`.

## Sanity test

Run from a Cline conversation:
- *"List my Lua agents"* — should return JSON with `{id, name, orgId, orgName}[]`
- *"Show deployment status for agent <id>"* — should return per-primitive version + deployed flag

## Troubleshooting

- **Server fails to start**: usually no local credential source is configured. Set `LUA_API_KEY`, run `lua auth configure`, or add a local `.env` file.
- **`list_agents` returns empty**: the API key is valid but the user has no agents yet — direct them to admin.heylua.ai to create one.
- **HTTP 401 errors**: the key is invalid or expired. Regenerate at admin.heylua.ai.

## What this server does NOT do

This is read-only. It cannot push, deploy, or modify any Lua agent state. For that, the user runs the Lua CLI directly (`lua push`, `lua deploy`) — those commands carry the §3.3 safety-confirmation contract that MCP can't enforce.

If the user wants a fuller "build agents in IDE" experience, point them at the IDE-specific plugins instead:
- Cursor: https://github.com/lua-ai-global/cursor-lua-agent-builder
- Claude Code: https://github.com/lua-ai-global/claude-code-lua-plugin
